---
layout: post
title: "Emitting events with database"
categories: microservices
---

In this article, you will find information on:
* how to guarantee delivery of event combined with database interactions. 

# Introduction
As a developer in many situations I need to execute some business logic and save results in database as well as emit event (e.g. Kafka event).
Assumptions:
* Database transaction need to be as quick as possible. I don't want to make any external calls inside transaction. Long transactions can degradate performance of database and extends lock time.
* I want to be sure that both database and stream will end-up in proper state: database transaction need to be committed and event need to be emitted - [two-phase commit problem](https://en.wikipedia.org/wiki/Two-phase_commit_protocol)
* Event need to be `delivered at-least-once` (it is completely fine to deliver event many times).
* Order of event is important inside partition only.

# Solution
## Happy path
In the solution we will use advantage of database transactions.

Algorithm:
1. Open database transaction.
2. Execute business logic 
3. Insert `intention` of emitting event to database. I do not specify what is `intention`, it can be: 
    * marshalled whole event and inserted to special table, 
    * inserted table row, base on which it would be possible to build event in deterministic way (easy when data are immutable).
5. Commit transaction.
6. Outside database transaction, send events.
7. Open database transaction.
8. Delete intentions of emitting event that were already emitted (you can also mark it as sent).
9. Commit transaction.

<figure>
  <img src="/assets/2019-12-01-emitting-events-with-db/happy_path.png" alt="Emit events happy path"> 
  <figcaption>Emit event happy path</figcaption>
</figure>

# Recover state
The algorithm above will work in happy path. But we want our system to recover from different situations:
* first database transaction will fail - in this scenario state of database will not change. Event will not be emitted as well. No action needed. 
* communication with stream will fail - in this scenario we need to introduce some recovery mode to send message again.
* second database transaction will fail - in this scenario we need to introduce some recovery mode to send message again and delete intention. 


## Transaction committed, but message not send 
One of the case is that transaction with logic was committed, intention to emit event is in database, but sending of event failed. 
In this scenario we can have at least 2 solutions: 
1. if we emit events frequently, we can emit event just before next event. 
The main difference is that we write to database intention of emitting event. 

1. Open database transaction.
2. Execute business logic 
3. Insert `intention` of emitting event to database. 
4. Commit transaction.
```diff
- 5. Open read inly transaction (it can increase performance)
- 6. Read all intentions of emitting events for given partition.
- 7. Commit read only transaction
```
7. Outside database transaction, send events.
8. Open database transaction.
9. Delete intentions of emitting event that were already emitted (you can also mark it as sent).
10. Commit transaction.

The algorithm has been presented below:
<figure>
  <img src="/assets/2019-12-01-emitting-events-with-db/recover_by_sending_events.png" alt="Emit events with another event"> 
  <figcaption>Emit events with another event</figcaption>
</figure>


2. if we do not events very frequently, we can create a special scheduler to send old 

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

# Summary
In this article I presented a safe way of emitting event together with database transaction.
If order of events is not important, you can easily simplify algorithm. 


4. Read from database all intentions of emitting events for given `business key` (e.g. account, user, where we want to have events in order).