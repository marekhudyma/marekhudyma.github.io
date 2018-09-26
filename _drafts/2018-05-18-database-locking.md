---
layout: post
title: "Postgres 10 Locking + Hibernate"
categories: sql
---

# Content 
In this post I will describe how to use:
* Optimistic read 
* Pessimistic read 
* Pessimistic write

# Disclaimer
I will talk only about: 
* Spring Boot + Hibernate, 
* Postgres DB,
* Isolation level - read committed,
* You can lock resources in many ways (not only on SQL level),
* There are other locking methods in SQL.


# How to use annotation @Lock in JpaRepository
```java
## Proper usage
Annotation @Lock should be added to 
@Repository
public interface EntityRepository extends JpaRepository<Entity, Long> {

    @Lock(LockModeType.PESSIMISTIC_WRITE) 
    Entity findOne(Long id);
}
```

```java
@Repository
public interface EntityRepository extends JpaRepository<Entity, Long> {

    @Lock(LockModeType.PESSIMISTIC_WRITE)  // IT WILL NOT WORK 
    default Card findOneAndLock(String id) {
        return findOne(id);
    }
}
```


LockModeType.NONE
RYSUNEK
select entity_.created as created2_2_ from entity entity_ where entity_.id=?

update entity_without_version set created=?, description=? where id=?


The row would have value set by transaction 2.



LockModeType.OPTIMISTIC (synonym READ)
Add @Version field in your entity. Supported types:
int, Integer, short, Short, long, Long, java.sql.Timestamp
If there is a different version of @Version field in the database, ObjectOptimisticLockingFailureException is thrown. 
select entitywith0_.id ... from entity entity0_ where entity0_.id=?
update entity set created=?, description=?, version=? where id=? and version=?
The version has been updated: 
at the end of transaction
every time we call saveAndFlush



LockModeType.OPTIMISTIC (synonym READ)
RYSUNEK


LockModeType.OPTIMISTIC_FORCE_INCREMENT (synonym WRITE)
OPTIMISTIC_FORCE_INCREMENT works in the same way as OPTIMISTIC but it increment version value every time.


LockModeType.PESSIMISTIC_READ
Hibernate: select entity0_.id as id1_2_0_, entity0_.created as created2_2_0_, entity0_.description as descript3_2_0_ from entity entity0_ where entity0_.id=? for share

Other transactions may concurrently read the entity, but cannot concurrently update it (if both use “select for share”).
There is a deadlock if two transactions use PESSIMISTIC_READ and try to modify the row. 
Random transaction interrupted with this exception (no matter which one started first). 
You can still read data using classical repository (without any locking strategy). 

LockModeType.PESSIMISTIC_READ
RYSUNEK

LockModeType.PESSIMISTIC_WRITE
Other transactions cannot concurrently read or write the entity (if both use “select for update”).

You can still read data using classical repository (without any locking strategy). 

select entity0_.id as id1_2_0_, entity0_.created as created2_2_0_, entity0_.description as descript3_2_0_ from entity entity0_ where entity0_.id=? for update


LockModeType.PESSIMISTIC_WRITE
RYSUNEK

MIX - PESSIMISTIC_WRITE + OPTIMISTIC
You can mix PESSIMISTIC_WRITE + OPTIMISTIC 


Add version field and use PESSIMISTIC_WRITE


https://github.com/marekhudyma/database














