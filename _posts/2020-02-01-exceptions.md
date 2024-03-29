---
layout: post
title: "Usage of exceptions in business applications"
featured: false
author: marek
categories: Java,Exceptions
type: post
image: '/assets/2020-02-01-exceptions/java_the_complete_reference_tenth_edition.jpg'
comments: false
---

In this article, you will find information on:
* Usage of exceptions in business applications

# Introduction
When I write a backend application, usually by using Spring Boot, I see that logical flow is done by exceptions. 
Imagine that HttpController calls AccountService to update some account information. Method signature can look like that:
```Account update(UpdateAccount data);```
For sure it can handle happy a path. But what if we have problems like:
* account doesn't exist,
* etag put in header does not match, 
* account is blocked and we cannot update some information
Usually developers thrown `RuntimeException` of a given type. These exceptions can have `HTTP annotations`, e.g.:

```java
@ResponseStatus(value = HttpStatus.NOT_FOUND)
public class ResourceNotFoundException extends RuntimeException {
}
```

So, in this scenario, we are mixing `service`  and `HTTP` layers. By definition in multilayer application, layer below (here exception from service) shouldn't have access to layer above (HTTP annotations).
We can also add `ExceptionHandler`, e.g.:
```java
@RestControllerAdvice(AccountController.class})
public class ApiExceptionHandling {
   // exception handlers 
}
```
But which exceptions should I handle then? To make it proper, we should read the full body of the service method. Also, in the future, while changing service method, I should make sure that all new cases are handled by HTTP layer. 

In this article I will explain my opinion in this topic. 
Before I came to the final conclusion, I would like to go back to `define Exception` and explain `Principle of the least astonishment`.

# Exceptions
In considering which definition of Exception is the best and which one is most trusted by Java developers, I decided to use one from book [Java: The Complete Reference, Tenth Edition](https://www.oreilly.com/library/view/java-the-complete/9781259589348/)

<figure>
  <img src="/assets/2020-02-01-exceptions/java_the_complete_reference_tenth_edition.jpg" alt="Java the Complete Reference, Tenth edition"> 
  <figcaption>Java the Complete Reference, Tenth edition</figcaption>
</figure>

On page 217 we can read:
> An exception is an abnormal condition that arises in a code sequence at run time. In other words, an exception is a run-time error. 

On page 236 we can read:
> Java's exception-handling statements should not be considered a general mechanism for nonlocal branching. 
> If you do so, it will only confuse your code and make it hard to maintain. 

What it means is that situation when we throw an exception needs to be `abnormal`. Is the lack of an Account in a database `abnormal` situation? For me, no. I can easily predict this kind of situation and avoid exceptions. 


# Principle of least astonishment
For me, `Principle of least astonishment` is a very important principle in IT. 

Let's read from [Wikipedia](https://en.wikipedia.org/wiki/Principle_of_least_astonishment)
> The principle of least astonishment (POLA), also called the principle of least surprise (alternatively a "law" or "rule") applies to user interface and software design.
> A typical formulation of the principle, from 1984, is: "If a necessary feature has a high astonishment factor, it may be necessary to redesign the feature."
> More generally, the principle means that a component of a system should behave in a way that most users will expect it to behave; the behavior should not astonish or surprise users. 

One of the most famous examples of breaking Principle of least astonishment is the standard C++ remove() function template.
When you call it on collection, I would expect to remove element from it. But what it does is reordering not "removed" elements at the head of the collection and returns pointer to the new end of collection. The size of the collection does not change. [Documentation of remove method](http://www.cplusplus.com/reference/algorithm/remove/).

In this context, for me, it is pretty clear that throwing usually undocumented RuntimeExceptions from the method looks like breaking `POLA`. 


# Arguments to not throw business exceptions 

There are many reasons why throwing business exceptions can be bad: 
* breaking `Principle of least astonishment`, as described above;
* Exceptions can be used as sophisticated GOTO statements;
* Exceptions pollute your application logs. As a developer, I would like to have notification when my application throws some exception, but also, I don't want to have false positive alerts;
* Usage of exceptions makes your code more difficult to read, understand and maintain;
* Modern programing languages can easily handle control flow without usage of exceptions;
* Exceptions degrade the speed of an application (throwing exceptions is one of the most expensive operations in Java).

# Example of control flow without exceptions
For an  example of a solution, I will show the Spring Http Controller method that calls `AccountService` update method with signature: 

```java
Result<Error, Account> execute(AccountUpdate accountUpdate);
```

It will return `Result` object with the Account inside for a happy path and `Result` in an enumeration `Error` if something went bad. Let's see an example:

```
public Result<Error, Account> execute(AccountUpdate accountUpdate) {
    return accountRepository.findById(accountUpdate.getId()).map(account -> {
        account.setScoring(accountUpdate.getScoring());
        return Result.ok(accountRepository.save(account));
    }).orElse(Result.fail(Error.ACCOUNT_NOT_FOUND));
}
```

The response can be mapped in nice way, with a `new switch expression` introduced in [Java 13](https://docs.oracle.com/en/java/javase/13/language/switch-expressions.html).
This structure will make you think about all corner cases and `new switch expression` will force you to map all of them. 

<pre><code>
@PatchMapping(value = "/accounts/{accountId}", consumes = PATCH_HEADER)
public ResponseEntity<?> patch(@PathVariable UUID accountId,
  @Valid @RequestBody PatchAccountDto patchAccountDto,
  @RequestHeader(name = HttpHeaders.ETAG) String etag) {

  return updateAccountCommand
    .execute(converter.convert(accountId, patchAccountDto.scoring(), etag))
<b>    .map(error -> switch (error) {
        case ACCOUNT_NOT_FOUND -> ResponseEntity.notFound().build();
    },
    account -> ResponseEntity.noContent().build());</b>
}
</code></pre>

# Database exceptions inside transaction

There is an important issue with `Spring transactions`.
Let's say there is a method annotated with the `@Transactional` annotation.
Some repositories can throw an exception, e.g. `DataIntegrityViolationException`.
Your code can catch it, but anyway, your whole transaction will be marked to be rolled back.

```
@Transactional 
public void create(Account account) {
    try {
        accountRepository.saveAndFlush(account);
    } catch (DataIntegrityViolationException ignore) {
        // do nothing 
    }
    otherRepository.saveAndFlush(OtherEntity.builder()); // <-- exception
}
```
In this situation the exception will be thrown: 
```
org.springframework.orm.jpa.JpaSystemException: could not execute statement; nested exception is org.hibernate.exception.GenericJDBCException: could not execute statement
```

# Summary
In this article, I provided arguments an why the usage of business exceptions is bad and how it can be replaced with other code structures.
