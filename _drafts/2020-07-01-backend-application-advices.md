---
layout: post
title: "Application style"
categories: convention
---

After years 
The purpose of this article is to present a vision how to create a backend application in Spring Boot. 
Disclaimer: This post is pretty long and detailed. I wanted to cover as many technical aspects as possible. I am also aware that there are many aspects which I didn't mention.  

Capitan obvious. 

Technical ssumptions: 
* used Java 14 with flag `--enable-preview`,
* used Spring Boot 2.3.2.RELEASE

# Assumptions
Main logical assumptions: 
* exceptions are not thrown only in a situation when it is not possible to write code in a different way, 
* application is written in a style of CQRS,
* follow Domain-driven design principles. 

## Exceptions 
In the article [Usage of exceptions in business applications](../_posts/2020-02-01-exceptions.md) I explained why controlling the flow of execution based on exceptions is not a good idea and presented alternative solution. 
(Disclaimer: exceptions are still good in exceptional situations).

## Command query responsibility segregation
In this article: [Command query responsibility segregation](../_posts/2020-03-01-CQRS.md) I explained the idea of using `Commands` and `Queries`.

# Code

## Data Transfer Object
The main properties of Data Transfer Objects
* It should just transport data,
* should be simple, without any logic inside (e.g. transforming to `entity`),
* be immutable.
* should make simple validation (without accessing resources like database, service layer),
* in never Java version, use `record` classes.

Example in Java 14 with flag `--enable-preview`,

## Validation
Validation should we be done by annotations from package: `javax.validation.constraints.*`.
You can also add `custom validators`, but never, ever call service layer / repositories inside. If you need to do it, make it in Service layer.

## API first 
API fist rule is very important for me 

https://opensource.zalando.com/restful-api-guidelines/


## Controllers 
Controllers main tasks are 
* handle HTTP call, 
* make simple validation of `DTO`,
* convert `DTO` to the object accepted by Service layer.
   * Service should not use object from layer above, so `DTO`s should not be passed to services directly as `DTO`s,
   * Controller can pass argument to the method, e.g. `service.call(argument1, argument2);`
   * Controller can convert `DTO` to Entity object (using Converter), 
   * Service can accept interface, `DTO` can implement this interface, than it can be passed directly to service, 
* handle response from `Service Layer` and map it to `HTTP` response. 
* Controller is not a part of your application to execute busieness logic. Keep it simple. 

```
@PatchMapping(value = "/accounts/{accountId}", consumes = PATCH_HEADER)
public ResponseEntity<?> patch(@PathVariable UUID accountId,
  @Valid @RequestBody PatchAccountDto patchAccountDto,
  @RequestHeader(name = HttpHeaders.ETAG) String etag) {

  return updateAccountCommand
    .execute(converter.convert(accountId, patchAccountDto.scoring(), etag))
    .map(error -> switch (error) {
        case ACCOUNT_NOT_FOUND -> ResponseEntity.notFound().build();
    },
    account -> ResponseEntity.noContent().build());
}
```
HTTP codes should not be returned by exceptions. Predictable exceptions should be handled by `RESPONSE`. 
Exception handler can we added exceptionally, for the exception types that cannot be handled in a different way, e.g.:
* MethodArgumentTypeMismatchException
* MethodArgumentNotValidException -
Use a proper `problem JSON` for handling errors, by following [standard](https://opensource.zalando.com/restful-api-guidelines/#176).
  
## Converter
Use Spring `org.springframework.core.convert.converter.Converter` interface to create Converters. It is a functional interface that can be tested in a separation. 

## Data Access
Most of the applications simple use `database`, but most of advices are valid to 


* Use as little business logic as possible
* Test it with IntegrationTests.
* When you 

poprawic @Transactional 
uzywaj Grapth cos tam 
import javax.persistence.NamedEntityGraphs; @NamedEntityGraphs



Model

    use Entity / Aggregate / Value Object patters
    Identified by id
    always use createdAt, updatedAt properties
    muttable
    no setters
    expose behavior
    Consider introducing key types

    @EmbeddedId
    @AttributeOverride(name = "value", 
    	column = @Column(name = "id"))
    private AccountId id;
    							


