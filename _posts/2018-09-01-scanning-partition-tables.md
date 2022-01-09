---
layout: post
title: "Scanning partitioned tables in PostgreSQL 10"
featured: false
author: marek
categories: sql
type: post
image: '/assets/2018-09-01-scanning-partition-tables/execution_plan_now.png'
comments: false
---

In this article, you will find information on: 
* How to scan partitioned tables in Postgres using function `now()`.

# Scanning partitioned tables 
In my application, I wanted to get rows older than the current time. 
I executed the explain plan for the query and I realized that the execution plan did not eliminate unnecessary partitions: 
```
EXPLAIN SELECT * FROM transactions WHERE created < now();  -- it is 2018-08-01
```
<figure>
  <img src="/assets/2018-09-01-scanning-partition-tables/execution_plan_now.png" alt="Execution plan for now function"> 
  <figcaption>not optimal execution plan</figcaption>
</figure>

I was surprised that DB didn't make this query optimal. Execution with the hardcoded value returned optimal execution plan:
```
EXPLAIN SELECT * FROM transactions WHERE created < '2018-01-01 00:00:00'
```
<figure>
  <img src="/assets/2018-09-01-scanning-partition-tables/execution_plan_static_date.png" alt="Execution plan for static date"> 
  <figcaption>optimal execution plan</figcaption>
</figure>

The reason is that function `now()` is not immutable. 
Constant values for the timestamps in the WHERE clause of the query work as expected but if the comparison is against a non-immutable function (such as now(), current_time, current_date, clock_timestamp(), etc.) then the query optimizer won’t be able to narrow down the scan to the relevant partitions, since it doesn’t know in which one the function value will fall into at runtime. It will, therefore, fall back to a full scan of all partitions removing the benefit of partitioning.

With this knowledge, I defined the function that returns `immutable now` :
```
CREATE FUNCTION immutable_now() RETURNS  TIMESTAMP WITH TIME ZONE AS $$
    SELECT now();
$$ LANGUAGE sql IMMUTABLE;
```
It seemed to work correctly! Rows were returned correctly, but suddenly it stopped returning newer rows.
The reason is that PostgreSQL started to `cache the results of the immutable_now` function.
<figure>
  <img src="/assets/2018-09-01-scanning-partition-tables/flow_diagram.png" alt="Flow diagram"> 
  <figcaption>PostgreSQL started to cache the result of the immutable_now function.</figcaption>
</figure>

# Reason 
Documentation on [function immutability](https://www.postgresql.org/docs/current/static/sql-createfunction.html):
_"IMMUTABLE indicates that the function [...] always returns the same result when given the same argument values; that is, it does not [...] use information not directly present in its argument list. If this option is given, any call of the function with all-constant arguments can be immediately replaced with the function value."_
`
and again in the documentation on function [volatility categories](https://www.postgresql.org/docs/current/static/xfunc-volatility.html)
_"Labeling a function IMMUTABLE when it really isn't might allow it to be prematurely folded to a constant during planning, resulting in a stale value being re-used during subsequent uses of the plan. This is a hazard when using [...] function languages that cache plans (such as PL/pgSQL)."_

# Solution 
The solution was to add random UUID as an argument, that will be ignored. PostgreSQL will not cache this function: 
```
CREATE FUNCTION immutable_now(ignored uuid) RETURNS TIMESTAMP WITH TIME ZONE AS 
$$
    SELECT now();
$$ LANGUAGE sql IMMUTABLE;
```
The invocation will look like that: 
```
EXPLAIN SELECT * FROM transactions 
WHERE created < immutable_now(uuid_generate_v4());
```

