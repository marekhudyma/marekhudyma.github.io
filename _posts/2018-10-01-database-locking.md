---
layout: post
title: "Database locking"
featured: false
author: marek
categories: sql
type: post
image: '/assets/2018-10-01-database-locking/transactions_without_locking.png'
comments: false
---

In this article, you will find information on: 
* Optimistic read, 
* Pessimistic read, 
* Pessimistic write.

I will cover the topic in context of:
* Hibernate with Spring, 
* PostgreSQL database (but it works in the same way on MySQL), 
* Isolation level - read committed,
 
Example of the working Java/SQL code you can find [here](https://github.com/marekhudyma/db-locking).

# Why do you need to lock a resource in a database?
Let's say that you have two transactions that are overlapping:
* Transaction 1 reads row, 
* Transaction 2 reads row,
* Both transactions are modifying one column in the row, 
* Transaction 1 commits the changes
* Transaction 2 commits the changes and **overwrites** the changes from transaction 1. 
<figure>
  <img src="/assets/2018-10-01-database-locking/transactions_without_locking.png" alt="Transactions without locking"> 
  <figcaption>Transactions without locking</figcaption>
</figure>
It can happen in many scenarios. The very common problem are two HTTP requests or queue consumptions coming one after another.

# How to add locking 
To add locking to your JpaRepository, you need to annotate the **interface method** with @Lock annotation. Here is an example of PESSIMISTIC_WRITE:
```
@Repository
public interface EntityRepository extends JpaRepository<Entity, Long> {

    @Lock(LockModeType.PESSIMISTIC_WRITE) 
    Optional<Entity> findById(Long id);
}
```
If you annotate the method with body it will not work: 
```
@Repository
public interface EntityRepository extends JpaRepository<Entity, Long> {

    @Lock(LockModeType.PESSIMISTIC_WRITE)  // IT WILL NOT WORK 
    default Card findOneAndLock(String id) {
        return findOne(id);
    }
}
```
Of course, you can create a native query with syntax `SELECT ... FROM TABLE FOR UPDATE`.

# Locking types

## None
LockModeType.NONE is a default behavior of JPA. It means there is no locking. 
Hibernate queries looks like this: 
```
select entity_.created as created2_2_ from entity entity_ where entity_.id=?
update entity_without_version set created=?, description=? where id=?
```

## Optimistic locking 
To use optimistic locking (LockModeType.OPTIMISTIC, synonym LockModeType.READ):
* Annotate the method with @Lock(LockModeType.PESSIMISTIC_WRITE)
* Add @Version field in your entity. Supported types:
    * int, 
    * Integer, 
    * short, 
    * Short, 
    * long, 
    * Long, 
    * java.sql.Timestamp

If there is a different version of the @Version field in the database, `ObjectOptimisticLockingFailureException` is thrown. 
<figure>
  <img src="/assets/2018-10-01-database-locking/transactions_optimistic_locking.jpg" alt="Transactions with optimistic locking"> 
  <figcaption>Transactions with optimistic locking</figcaption>
</figure>

Hibernate queries looks like this: 
```
select entitywith0_.id ... from entity entity0_ where entity0_.id=?
update entity set created=?, description=?, version=? where id=? and version=?
```

The good source of information about optimistic lock is [here](https://www.baeldung.com/jpa-optimistic-locking).

### OPTIMISTIC_FORCE_INCREMENT 
OPTIMISTIC_FORCE_INCREMENT – it obtains an optimistic lock the same as OPTIMISTIC and additionally increments the version attribute value.

## Pessimistic read
To use pessimistic read (LockModeType.PESSIMISTIC_READ):
* Annotate the method with @Lock(LockModeType.PESSIMISTIC_READ).

It means that other transactions may concurrently read the entity, but cannot concurrently update it (if both use `select for share`).

Hibernate queries looks like this: 
```
Hibernate: select entity0_.id as id1_2_0_, entity0_.created as created2_2_0_, 
entity0_.description as descript3_2_0_ 
from entity entity0_ where entity0_.id=? for share
```

There is a deadlock if two transactions use PESSIMISTIC_READ and try to modify the row. 
Random transaction interrupted with this exception (no matter which one started first). 
You can still read data using a classical repository (without any locking strategy). 

<figure>
  <img src="/assets/2018-10-01-database-locking/transactions_pessimistic_read.jpg" alt="Transactions with pessimistic read"> 
  <figcaption>Transactions with pessimistic read</figcaption>
</figure>

Personally, I find the pessimistic read to be the least useful locking type.  

## Pessimistic write
To use pessimistic write (LockModeType.PESSIMISTIC_WRITE):
* Annotate the method with @Lock(LockModeType.PESSIMISTIC_WRITE).

Other transactions cannot concurrently read or write the entity (if both use `select for update`).
You can still read data using a classical repository (without any locking strategy). 

Hibernate queries looks like that: 

```
select entity0_.id as id1_2_0_, entity0_.created as created2_2_0_, 
entity0_.description as descript3_2_0_ from entity entity0_ where entity0_.id=? for update 
```

<figure>
  <img src="/assets/2018-10-01-database-locking/transactions_pessimistic_write.jpg" alt="Transactions with pessimistic write"> 
  <figcaption>Transactions with pessimistic write</figcaption>
</figure>

## Locking MIX 
You can mix PESSIMISTIC_WRITE + OPTIMISTIC. Add version field and use PESSIMISTIC_WRITE

# Alternative solutions 
You may lock resources in many ways, for example: 
* in Redis (for example using [Redisson](https://redisson.org/))
* files.

# Summary 
Locking resources is not a trivial task. Database locking is easy to implement and can be applied in many enterprise applications.

| Lock type             |                                                                                      |
| --------------------- |:------------------------------------------------------------------------------------:|
| optimistic locking    | you can read, but the exception is thrown during write if `version` field doesn't match. | 
| pessimistic read      | you can read entity, but you cannot write                                            |
| Pessimistic write     | you lock entity for reading                                                          |  
| mix                   | you cannot read and write if `version` field doesn't match.                          |