---
layout: post
title: "Microsoft SQL Server trailing spaces"
featured: false
author: marek
categories: SQL
type: post
image: '/assets/2024-05-01-microsoft-sql-server-trailing-spaces/microsoft-sql-server-logo.png'
comments: false
---

<figure> 
    <center>
      <img src="/assets/2024-05-01-microsoft-sql-server-trailing-spaces/microsoft-sql-server-logo.png" alt="Microsoft SQL Server logo" />
    </center>
</figure>

# Introduction
In this article, we will delve into the unexpected behavior of the equals (=) operator in Microsoft SQL Server.
Let's explore some scenarios where the equals operator might not work as intuitively as one might expect.

## Microsoft SQL Server equals operator

In Microsoft SQL Server, the behavior of the equals (=) operator can be quite surprising, 
especially when it comes to handling trailing spaces. 
This quirk can lead to unexpected results in your queries. Let's examine a specific scenario to illustrate this.

Imagine we have a table `my_table` with a primary key of type `VARCHAR`. Consider the following queries:

```SQL
SELECT * FROM my_table WHERE id = 'ABC';
SELECT * FROM my_table WHERE id = 'ABC ';
```

Both of these queries will return the row identified by id='ABC', despite the trailing space in the second query. 
This behavior occurs because SQL Server ignores trailing spaces when comparing VARCHAR values.

## Microsoft SQL Server documentation

In the [official documentation of Microsoft SQL Server](https://learn.microsoft.com/en-us/sql/t-sql/language-elements/string-comparison-assignment?view=sql-server-ver16#d-string-comparison-with-spaces),
the behavior of the equals (=) operator is described as follows:
```
String comparison using the = operator assumes that both strings are identical.
```
However, the term "identical" can be misleading, particularly when it comes to handling trailing spaces.
In the dedicated section on ["String comparison with spaces"](https://learn.microsoft.com/en-us/sql/t-sql/language-elements/string-comparison-assignment?view=sql-server-ver16#d-string-comparison-with-spaces)
the same documentation clarifies that SQL Server ignores trailing spaces in string comparisons.
This means that for VARCHAR values, a string with trailing spaces will be considered equal to the same string without those spaces.

## Deeper dive 
To illustrate the behavior of the equals (=) operator, let's create a temporary table with a single row. 
This demonstration will help clarify how SQL Server handles trailing spaces in string comparisons.

```SQL
CREATE TABLE #MyTable (my_column VARCHAR(50));
INSERT INTO #MyTable VALUES ('text');
```

```SQL
SELECT * FROM #MyTable WHERE my_column = 'text';  --returns row
SELECT * FROM #MyTable WHERE my_column = 'text '; --returns row

SELECT * FROM #MyTable WHERE my_column IN ('text');  --returns row
SELECT * FROM #MyTable WHERE my_column IN ('text '); --returns row

SELECT * FROM #MyTable WHERE my_column <> 'text';  --no results
SELECT * FROM #MyTable WHERE my_column <> 'text '; --no results

SELECT * FROM #MyTable WHERE my_column LIKE 'text'; --returns row
SELECT * FROM #MyTable WHERE my_column LIKE 'text '; --no results
SELECT * FROM #MyTable WHERE my_column LIKE 'text%'; --returns row

```

To summarize the consequences of how SQL Server handles string comparisons with the equals (=) operator:

* `Identical Text`: It will obviously return results for truly identical text.
* `Trailing Spaces`: Surprisingly, it also returns results for text with trailing spaces.
* `IN Syntax`:The IN syntax behaves the same way as the equals operator, matching both identical text and text with trailing spaces.
* `Not Equals (<>)`: The not equals (`<>`) operator does not return results for either identical text or text with trailing spaces.
* `LIKE Statement`: The LIKE statement returns results for truly identical matches but does not return results for text with trailing spaces.

### Foreign key

It's important to highlight that in Microsoft SQL Server, a foreign key constraint does not prevent the insertion 
of an id with trailing spaces. This means that even if you have a foreign key relationship set up between two tables,
SQL Server will still allow the insertion of a value with trailing spaces into the foreign key column.
`This can lead to unexpected issues in data integrity and consistency`.

## Spring Boot and Hibernate 

When using a Spring Boot application with Hibernate and Microsoft SQL Server,
you might encounter unexpected behavior related to trailing spaces in query parameters.

Consider a scenario where there is a row in the database with id = 'ABC'. 
If a query allows for trailing spaces (such as a value coming from an endpoint), 
Hibernate will execute a standard query like this:

```SQL
select ct_0.id (...) from my_table ct_0 where ct_0.id=?
```

Surprisingly, Hibernate may not read the id field from the database but instead take it from the cache. 
Consequently, this can result in an entity having `id='ABC '` — an incorrect value with trailing spaces.

This behavior can lead to significant issues, as incorrect values are used in your application. 
It's crucial to ensure that input values are sanitized and trailing spaces are handled appropriately 
to maintain data integrity and consistency.


# Proposed solution

To safeguard your database from misleading values, such as trailing spaces, consider implementing the following action points in your Spring Boot application:

* `Validate Inputs` - Ensure that incorrect inputs (e.g., those with trailing spaces) do not enter the system. 
In a Spring Boot application, you can add validation to the endpoints to achieve this. 
You may simply add a custom annotation to reject invalid requests. 

```Java
public class NoTrailingSpaces 
    implements ConstraintValidator<NoTrailingSpacesConstraint, String> {
    @Override
    public boolean isValid(String value,
                           ConstraintValidatorContext context) {
        return (value != null && !value.isEmpty() && value.equals(value.trim()));
    }
}
```

* `Validation During Data Saving` - Add validation to validator to entities to prevent saving data with trailing spaces.
In fact, you may use the same validator in the entity and DTO request object. 

* `Validate Entities After Loading` - In some cases, the application should fail upon loading incorrect data. 
To enforce this, execute entity validation in the @PostLoad entity event.

```Java
@Entity
public class MyEntity {
    @PostLoad
    public void validate() {
        if (id != null && !id.equals(id.trim())) {
            throw new RuntimeException("Trailing spaces found in id");
        }
    }
   // other fields ... 
}
```


By implementing these validation strategies, you can maintain data integrity and consistency in your Spring Boot applications, ensuring that trailing spaces do not cause unexpected issues.

# Summary
In this article, we explored the surprising behavior of the equals (=) operator in Microsoft SQL Server.
We highlighted the potential risks associated with this behavior, such as issues with data integrity and correctness.
By incorporating these validation strategies, you can effectively maintain data integrity and consistency, 
preventing the unexpected behavior of the equals operator from causing significant issues in your database.
