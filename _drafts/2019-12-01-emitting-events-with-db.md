---
layout: post
title: "Emitting events with database"
categories: microservices, stream
---

In this article, you will find information on:
* how to guarantee delivery of event combined with database interactions. 

# Introduction
As a developer in many situations I need to execute some business logic and save results in database as well as emit event (e.g. Kafka event).
At the beginning it looks like easy task, but it gets more and more complicated when you go into details. 
What make it complicated is [two-phase commit problem](https://en.wikipedia.org/wiki/Two-phase_commit_protocol). You need to set two sources in a proper state: database and stream. You want to either make success or fail in both places. 


Assumptions of a solution:
* Database transaction need to be as quick as possible. I don't want to make any external calls inside transaction with business logic.
* I want to be sure that both database and stream will end-up in proper state: everything fail or everything finish with success - database transaction need to be committed and event need to be emitted - [two-phase commit problem](https://en.wikipedia.org/wiki/Two-phase_commit_protocol)
* Event need to be `delivered at-least-once` (it is completely fine to deliver event many times).
* Order of event is important inside partition only.

# Solution
## Happy path
Below I describe `happy path` solution where we do not care two-phase commit problem. I


In the solution we will use advantage of database transactions.

Algorithm:
1. Open database transaction.
2. Execute business logic 
3. Insert `intention` of emitting event to database. I do not specify what is `intention`, it can be: 
    * marshalled whole event and inserted to special table, 
    * inserted table row, base on which it would be possible to build event in deterministic way (easy when data are immutable).
4. Commit transaction.
5. Outside database transaction, send events.
6. Open database transaction.
7. Delete intentions of emitting event that were already emitted (you can also mark it as sent).
8. Commit transaction.

<figure>
  <img src="/assets/2019-12-01-emitting-events-with-db/happy_path.png" alt="Emit events happy path"> 
  <figcaption>Emit event happy path</figcaption>
</figure>

# Recover state
The algorithm above will work in happy path. But we want our system to recover from different situations:
* **first database transaction fails** - in this scenario state of database will not change. Event will not be emitted as well. **No action needed.**
* **emitting event fails** - in this scenario we need to introduce some recovery mode to emit event again.
* **second database transaction fails** - in this scenario we need to introduce some recovery mode to delete intentions of emitting events.

## Emitting event fails
In this case transaction with logic was committed, intention to emit event was saved in database, but sending of event failed. Here are two solutions for this problem.

### Frequent events 
If we emit events frequently for all partitions, we can emit old events just before current one. 
Before emitting current event we need to check if in database we have some old and send it.

Detail algorithm: 
1. Open database transaction.
2. Execute business logic 
3. Insert `intention` of emitting event to database. 
4. Commit transaction.
5. <span style="color:red;">Open read only transaction (it can increase performance).</span>
6. <span style="color:red;">Read all intentions of emitting events for given partition.</span>
7. <span style="color:red;">Commit read only transaction.</span>
8. Outside database transaction, send events.
9. Open database transaction.
10. Delete intentions of emitting event that were already emitted (you can also mark it as sent).
11. Commit transaction.

The algorithm has been presented below:
<figure>
  <img src="/assets/2019-12-01-emitting-events-with-db/recover_by_sending_events.png" alt="Emit events with another event"> 
  <figcaption>Emit events with another event</figcaption>
</figure>

### Rare events
If we do **not** emit events very frequently, we can create a special scheduler to emit old events.

Every interval of time (depends on us) scheduler need to execute algorithm.
1. Open database transaction.
2. Read old and not sent events for a given partition key (e.g. account, user).
3. Commit transaction.
4. Outside database transaction, send events.
5. Open database transaction.
6. Delete intentions of emitting events that were already emitted (you can also mark it as sent).
7. Commit transaction.

<figure>
  <img src="/assets/2019-12-01-emitting-events-with-db/emit_part2.png" alt="Emit events - scheduler"> 
  <figcaption>Visualisation of scheduler path of emitting events</figcaption>
</figure>

## Second database transaction fails
It can happen that second transaction fails - after successfully emitting event, we fail to delete intention from database.
In this scenario our auto-recovering mechanism will handle the situation. In both cases (described in frequent events and rare events) algorithms will emit again events. 
It is not a problem since events can be delivered 1 or more times. 

# Summary
In this article I presented a safe way of emitting event together with database transaction.
If order of events is not important, you can easily simplify algorithm. 