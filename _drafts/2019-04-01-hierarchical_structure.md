---
layout: post
title: "Hierarchical structure"
categories: sql
---

In this article, you will find information on:
* how to model hierarchical structures

# Problem description 
The are many ways to model hierarchical structure. A good example can be discussion forum where users can reply to each other. 

<figure>
    <img src="/assets/2019-04-01-hierarchical_structure/hierarchical_structure.png" alt="hierarchical structure"> 
</figure>

There are many advantages and disadvantages of all solutions and non of them is better or worse in all cases. You need to find which solutions fits to your case. 

# Introduction 
This article was written based on book "SQL Antipatterns Avoiding the Pitfalls of Database Programming" by Bill Karwin and my personal experience. 

## Adjacency List anti-pattern 
In adjacency list  solution each comment references the comment to which it replies.
Table structure can look like: 
```
CREATE TABLE comments (
    id SERIAL   PRIMARY KEY,
    parent_id   BIGINT,
    comment     TEXT NOT NULL,
    author      TEXT NOT NULL,
    FOREIGN KEY (parent_id) REFERENCES comments (id)
);
```
Let's insert data into table:
```
INSERT INTO comments (id, parent_id, author, comment) VALUES
(1,  NULL, 'Frank', 'What is the cause of the problem ?'),
(2,  1,    'Alex',  'I think it is because empty reference'),
(3,  2,    'Frank', 'I did not check it'),
(4,  1,    'Kate',  'We need to validate input data'),
(5,  4,    'Alex',  'Yes, there is a mistake'),
(6,  4,    'Frank', 'Indeed, add validation code'),
(7,  6,    'Kate',  'It helped');
(9,  NULL, 'Kate',  'Is it interesting post?');
(10, 9,    'Frank',  'Yes, defenetly');
```
## Reading subtree 
It became clear that it is difficult to retrieve only some part of the tree with one query. 

To read second level of subtree with id 1, we need to make self OUTER JOIN.
```
SELECT c2.id, c2.parent_id, c2.author, c2.comment
FROM comments c1
LEFT OUTER JOIN comments c2 ON c2.parent_id = c1.id
WHERE c1.id = 1
```

<figure>
    <img src="/assets/2019-04-01-hierarchical_structure/adjacency_list_join.png" alt="self OUTER JOIN"> 
    <figcaption>Result of self OUTER JOIN</figcaption>
</figure>

To read third level of subtree with id 1, we need to make double self OUTER JOIN.
```
SELECT c3.id, c3.parent_id, c3.author, c3.comment
FROM comments c1
LEFT OUTER JOIN comments c2 ON c2.parent_id = c1.id
LEFT OUTER JOIN comments c3 ON c3.parent_id = c2.id
WHERE c1.id = 1
```
<figure>
    <img src="/assets/2019-04-01-hierarchical_structure/adjacency_list_join2.png" alt="double self OUTER JOIN"> 
    <figcaption>Result of double self OUTER JOIN</figcaption>
</figure>

Obviously this way of reading data will not fit to trees with dynamic depth. 

* There is no elegant way of reading subtree: 
1. execute SQL query for every row (that doesn't look like efficient solution),
2. read whole tree in memory (can be a solution for very small tree),
3. make self join if we know that tree will not be deeper that some level (I do not believe in this assumption at all) 

## Deleting subtree
To delete subtree you need to delete leafs elements, one by one to the top. 

## Adding new element
To add a new element you need to know the id of parent and insert row. 

## Deleting element 
Deletion leaf element is relatively easy, just delete row by id. 
Deletion of element inside the tree require relinking parent-child first.  

## Data consistency 
This solution uses database 

## How to recognize this anti-pattern? 
* when you her question about how many levels do you need to support ?
* You’re struggling to get all descendants or all ancestors of a node, without using a recursive query. 

# Path enumeration
The most important part of the solution is that we store the path to of the comment as file path in unix system in text field. 
Ids can be slash separated, eg: `1/2/3`.

In the book "SQL Antipatterns Avoiding the Pitfalls of Database Programming" it was mentioned as proper solution. 
My experience, hart and another book: "The Art of PostgreSQL" by Dimitri Fontaine find it as antipattern. 
The reason is a lack of usage of database data consistency mechanisms. 


Table structure can look like: 
```
CREATE TABLE comments (
    id          SERIAL   PRIMARY KEY,
    path        TEXT NOT NULL,
    comment     TEXT NOT NULL,
    author      TEXT NOT NULL,
    FOREIGN KEY (parent_id) REFERENCES comments (id)
);
```

Let's insert data into table:
```
INSERT INTO comments (id, path, author, comment) VALUES
(1, '1/',       'Frank', 'What is the cause of the problem ?'),
(2, '1/2/',     'Alex',  'I think it is because empty reference'),
(3, '1/2/3',    'Frank', 'I did not check it'),
(4, '1/4',      'Kate',  'We need to validate input data'),
(5, '1/4/5/',   'Alex',  'Yes, there is a mistake'),
(6, '1/4/6/',   'Frank',  'Indeed, add validation code'),
(7, '1/4/6/7/', 'Kate',   'It helped');
```

## Reading subtree 
To read subtree of the structure you can use LIKE query, eg.: 
```
SELECT * FROM comments c
WHERE c.path LIKE '1/4/%'
```
Results:
<figure>
    <img src="/assets/2019-04-01-hierarchical_structure/path_enumeration_select.png" alt="Path enumeration select of subtree"> 
    <figcaption>Path enumeration select of subtree</figcaption>
</figure>

The advantage is simplicity of query.  Disadvantage is the fact that the structure is in text field. 
Performance can be fixed by adding INDEX on this field. 

## Deleting subtree
You can easily delete data using LIKE statement, eg:
```
DELETE FROM comments c
WHERE c.path LIKE '1/4/%'
```
## Adding new element
To add a new row, you need to know the `path` of the parent, insert the row. After insert you know what is the `id` of the row and you can update `path` for your row.

## Deleting element 
Deletion leaf element is very easy by deleting the row.
Deletion element inside is more complicated because you need to rewrite all path of child elements. 

## Data consistency 
The `path` field doesn't use any database build in mechanisms of consistency. Is is possible to corrupt data inside table.  

# Nested Sets
The Nested Sets solution stores information with each node that pertains to the set of its descendants.

In each row there are two fields: `left_nodes` and `right_nodes` defined in a way:
* `left_nodes` is a number smaller than the number of all the children nodes,
* `right_nodes` is a number bigger than the number of all the children nodes.
**These numbers have no relation to the `id` values.**

One of the easiest way of assigning this values is by following a depth-first traversal of the tree, that was visualized below. 

<figure>
    <img src="/assets/2019-04-01-hierarchical_structure/hierarchical_structure_nested_sets.png" alt="nested_sets"> 
    <figcaption>Nested sets left and right nodes values</figcaption>
</figure>

Database structure:
```
CREATE TABLE comments (
    id          SERIAL   PRIMARY KEY,
    left_nodes  BIGINT NOT NULL,
    right_nodes BIGINT NOT NULL,
    thread_id   BIGINT NOT NULL,
    comment     TEXT NOT NULL,
    author      TEXT NOT NULL
);
```
Inserts needed:

```
INSERT INTO comments (id, left_nodes, right_nodes, thread_id, author, comment) VALUES
(1,  1, 14, 1, 'Frank', 'What is the cause of the problem ?'),
(2,  2,  5, 1, 'Alex',  'I think it is because empty reference'),
(3,  3,  4, 1, 'Frank', 'I did not check it'),
(4,  6, 13, 1, 'Kate',  'We need to validate input data'),
(5,  7,  8, 1, 'Alex',  'Yes, there is a mistake'),
(6,  9, 12, 1, 'Frank',  'Indeed, add validation code'),
(7, 10, 11, 1, 'Kate',   'It helped'),
(8,  1,  4, 2, 'Kate',   'Is it interesting post?'),
(9,  2,  3, 2, 'Kate',   'Yes, defenetly');
```

## Reading subtree 
Reading subtree you can achieve with query:
```
SELECT c2.*
FROM comments AS c1
JOIN comments as c2 ON c2.left_nodes BETWEEN c1.left_nodes AND c1.right_nodes
WHERE c1.id = 1 AND c2.thread_id = 1
```

<figure>
    <img src="/assets/2019-04-01-hierarchical_structure/nested_sets_select.png" alt="nested sets select"> 
</figure>


## Deleting subtree
One chief strength of the Nested Sets design is that when you delete a nonleaf node, its descendants are automatically considered direct children of the deleted node’s parents. Although the right and left numbers of each node shown in the illustration have values forming a continuous series and the difference is always one compared to adjacent siblings and parents, this is not necessary for the Nested Sets design to preserve the hierarchy. So when gaps in the values result from deleting a node, there is no interruption to the tree structure.

## Adding new element
When you insert a new node, you need to recalculate all the left and right values greater than the left value of the new node. This includes the new node’s right siblings, its ancestors, and the right siblings of its ancestors. It also includes descendants, if the new node is inserted as a nonleaf node.

## Deleting element 
Deletion leaf element is very easy by deleting the row.

## Data consistency 
It uses database mechanism to keep data consistent. 

# Closure Table

The Closure Table introduces two tables. First for storing comments and second for structure. 
Table comments with value looks like that:

```
CREATE TABLE comments (
    id          SERIAL   PRIMARY KEY,
    comment     TEXT NOT NULL,
    author      TEXT NOT NULL
);
```
Values:
```
INSERT INTO comments (id, author, comment) VALUES
(1, 'Frank', 'What is the cause of the problem ?'),
(2, 'Alex',  'I think it is because empty reference'),
(3, 'Frank', 'I did not check it'),
(4, 'Kate',  'We need to validate input data'),
(5, 'Alex',  'Yes, there is a mistake'),
(6, 'Frank',  'Indeed, add validation code'),
(7, 'Kate',   'It helped');
```
Second table stores two columns only: ancestor and descendant. 
One row in this table for each pair of nodes in the tree that shares an ancestor/descendant relationship, **even if they are separated by multiple levels in the tree**.

```
CREATE TABLE tree_paths (
    ancestor    BIGINT NOT NULL,
    descendant  BIGINT NOT NULL,
    PRIMARY KEY(ancestor, descendant),
    FOREIGN KEY (ancestor) REFERENCES comments(id),
    FOREIGN KEY (descendant) REFERENCES comments(id)
);
```

Values:
```
INSERT INTO tree_paths (ancestor, descendant) VALUES
(1,1),
(1,2),
(1,3),
(1,4),
(1,5),
(1,6),
(1,7),
(2,2),
(2,3),
(3,3),
(4,4),
(4,5),
(4,6),
(4,7),
(5,5),
(6,6),
(6,7),
(7,7)
```





## Reading subtree 



## Deleting subtree

## Adding new element
To add a new element, first insert a comment itself. Then insert path by: 
1. creating reference to itself
2. creating references to all ancestors (you can read it from parent node).

## Deleting element 
To delete a leaf node, for instance comment #7, delete all rows in tree_paths that reference comment #7 as a descendant.

## Data consistency 
It uses database mechanism to keep data consistent. 