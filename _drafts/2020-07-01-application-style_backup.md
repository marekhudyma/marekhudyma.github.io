---
layout: post
title: "Application style"
categories: convention
---

The purpose of this article is to present a vision how to create a backend application in Spring Boot. 
Disclaimer: This post is pretty long and detailed. I wanted to cover as many technical aspects as possible. I am also aware that there are many aspects which I didn't mention.  


Technical ssumptions: 
* used Java 14 with flag `--enable-preview`,
* used Spring Boot 2.3.2.RELEASE

# Assumptions 

Main logical assumptions: 
* exceptions are not thrown only in a situation when it is not possible to write code in a different way, 
* application is written in a style of CQRS,
* follow Domain-driven design principles. 

## Exceptions 
In the article [Usage of exceptions in business applications](2020-02-01-exceptions.md) I ment


## Command query responsibility segregation
Command query responsibility segregation (CQRS) applies the CQS principle by using separate Query and Command objects to retrieve and modify data, respectively.

[Command query responsibility segregation](2020-03-01-CQRS.md)

According to [Wikipedia](https://en.wikipedia.org/wiki/Command%E2%80%93query_separation)

> Command–query separation (CQS) is a principle of imperative computer programming [...]
> It states that every method should either be a command that performs an action, or a query that returns data to the caller, but not both. In other words, asking a question should not change the answer.

[CQRS](https://en.wikipedia.org/wiki/Command%E2%80%93query_separation#Command_query_responsibility_segregation)

> Command query responsibility segregation (CQRS) applies the CQS principle by using separate Query and Command objects to retrieve and modify data, respectively.

The main idea is to split business operations to `COMMANDS` and `QUERIES`.

# Implementation of Command

We can define all commands as objects which implements interface below:
```java
public interface Command<INPUT, RESULT> {
    RESULT execute(INPUT input);
}
```

Example implementation of `command` can look like:
```java
@Component
@RequiredArgsConstructor
public class CreateAccountCommand 
  implements Command<Account, Result<Error, Account>> {

    private final AccountRepository accountRepository;

    @Transactional
    public Result<Error, Account> execute(Account account) {
        return accountRepository.findById(account.getId())
                .map(a -> Result.fail(a, Error.ACCOUNT_ALREADY_EXIST))
                .orElseGet(() -> Result.ok(accountRepository.save(account)));
    }

    public enum Error {
        ACCOUNT_ALREADY_EXIST
    }

}
```

# Implementation of Query

We can define all `query` as objects which implements interface below (almost the same as `command`):
```java
public interface Query<INPUT, RESULT> {

    RESULT execute(INPUT input);

}
```

Example of implementation can look like:

```java
@Component
@RequiredArgsConstructor
public class GetAccountQuery 
  implements Query<AccountId, Result<Error, Account>> {

    private final AccountRepository accountRepository;

    @Transactional(readOnly = true)
    public Result<Error, Account> execute(AccountId accountId) {
        return accountRepository.findById(accountId)
                .map(Result::<Error, Account>ok)
                .orElse(Result.fail(Error.ACCOUNT_NOT_FOUND));
    }

    public enum Error {
        ACCOUNT_NOT_FOUND
    }
}
```


Advantages of usage of CQRS

    By using CQRS you have a clear separation between read and write operations. For example read-only database transactions you mark as read-only. E.g. by using annotation @Transactional(readOnly = true),
    All operations are separated that fit to Single responsibility principle.
    When performance is critical, with CQRS, you can optimize read and write sides independently,
    CQRS may simplify understanding of domain by dividing problem into the command and query parts,
    You can split work between two teams: one that is implementing commands, second implementing queries,
    When you already use the Event Sourcing, it combines nicely with CQRS.
    It is easy to find all occurrences of commands in the code. In fact, you can find all places where you modify data.




## Domain-driven design 



What will not be covered: 
* how to design proper API, [here](https://opensource.zalando.com/restful-api-guidelines/) you can find good guildeline. 


[Event sourcing architecture](2020-02-01-exceptions.md)






public interface Command<INPUT, RESULT> {

    RESULT execute(INPUT input);

}

public interface Query<INPUT, RESULT> {

    RESULT execute(INPUT input);

}

# Package structure 

# Web layer 

## Data Transfer Object
* `DTO`s should be simple, without any logic inside (e.g. transforming to `entity`). 
* It should just transport data 
* should make simple validation (without accessing resources like database)
* be immutable. 

Example in Java 14 with flag `--enable-preview`,
```
// TODO remove @JsonProperty when Jackson support records
public record CreateAccountDto(

        @NotNull
        @JsonProperty("id")
        UUID id,

        @NotEmpty
        @JsonProperty("name")
        String name,

        @NotNull
        @Min(0)
        @Max(100)
        @JsonProperty("scoring")
        Integer scoring
) {

}
```

//@Builder
//@RequiredArgsConstructor
//@Data
//public class CreateAccountDto {
//
//    @NotNull
//    private final UUID id;
//
//    @NotBlank
//    @Max(50)
//    private final String name;
//
//    @Min(0)
//    @Max(100)
//    private final int scoring;
//
//}

Validation 
Validation should we done by annotations from package: `javax.validation.constraints.*`. 

You can also add custom validators: 
@Constraint(validatedBy = CampaignLineLifetimeValidator.class)
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface ValidCampaignLineLifetime {

    String message() default "'Active from' must be before or equal 'Active till'.";

    Class<?>[] groups() default { };

    Class<? extends Payload>[] payload() default { };
}
---
@ValidCampaignLineLifetime
---
public class CampaignLineLifetimeValidator
    implements ConstraintValidator<ValidCampaignLineLifetime, CampaignLineItem> {


    @Override
    public void initialize(ValidCampaignLineLifetime validCampaign) {

    }

    @Override
    public boolean isValid(CampaignLineItem campaignLineItem, ConstraintValidatorContext constraintValidatorContext) {
        if (campaignLineItem.getActiveFrom() == null || campaignLineItem.getActiveUntil() == null) {
            return true;  // let this covered by other validators, so that we don't show stupid messages
        }
        return !campaignLineItem.getActiveFrom().isAfter(campaignLineItem.getActiveUntil());
    }
}
---
What is really important is that you execute only pretty simple logic without accessing external beans/resources like database (it would make testing much more difficult)


## Controllers 
Controllers main tasks is to: 
* handle http call, 
* make simple validation of `DTO` (e.g. using `@Valid` annotation),
* convert `DTO` to the object accepted by Service layer.
   * Service should not use object from layer above, so `DTO`s should not be passed to services directly as `DTO`s,
   * Controller can pass argument to the method, e.g. `service.call(argument1, argument2);`
   * Controller can convert `DTO` to Entity object (using Converter), 
   * Service can accept interface, `DTO` can implement this interface, than it can be passed directly to service, 
* handle response from `Service Layer` and map it to `HTTP` response. 
```
	final var result = removeBoardCommand.run(userId, boardId);
	return switch (result) {
	    case BOARD_NOT_FOUND -> ResponseEntity.notFound().build();
	    case PERMISSION_DENIED -> ResponseEntity.status(FORBIDDEN).build();
	    case ALREADY_DELETED -> ResponseEntity.status(CONFLICT).build();
	    case OK -> ResponseEntity.ok().build();
	};
```
HTTP codes should not be returned by exceptions. Predictable exceptions should be handled by `RESPONSE`. 
Exception handler can we added exceptionally, for the exception types that cannot be handled in a different way, e.g.:
* MethodArgumentTypeMismatchException
* MethodArgumentNotValidException - 
* Exception - 
Use a proper `problem JSON` for handling errors, by following [standard](https://opensource.zalando.com/restful-api-guidelines/#176).

@RestControllerAdvice(assignableTypes = {AccountController.class})
public class ApiExceptionHandler {

    @ExceptionHandler(MethodArgumentNotValidException.class)
    @ResponseStatus(value = HttpStatus.BAD_REQUEST)
    ProblemDto handleException(WebRequest request, MethodArgumentNotValidException cause) {
        return new ProblemDto(HttpStatus.BAD_REQUEST.name(),
                HttpStatus.BAD_REQUEST.value(),
                cause.getMessage());
    }

    @ExceptionHandler(MethodArgumentTypeMismatchException.class)
    @ResponseStatus(value = HttpStatus.BAD_REQUEST)
    ProblemDto handleException(WebRequest request, MethodArgumentTypeMismatchException cause) {
        return new ProblemDto(HttpStatus.BAD_REQUEST.name(),
                HttpStatus.BAD_REQUEST.value(),
                cause.getMessage());
    }

    @ExceptionHandler(Exception.class)
    @ResponseStatus(value = HttpStatus.INTERNAL_SERVER_ERROR)
    ProblemDto handleException(WebRequest request, Exception cause) {
        return new ProblemDto(HttpStatus.INTERNAL_SERVER_ERROR.name(),
                HttpStatus.INTERNAL_SERVER_ERROR.value(),
                cause.getMessage());
    }

}


# Converter
Use Spring `org.springframework.core.convert.converter.Converter` interface to create Converters. It is a functional interface that can be tested in a separation. 

@Service
public class CreateAccountDtoToAccountConverter implements Converter<CreateAccountDto, Account> {

    @Override
    public Account convert(CreateAccountDto source) {
        return Account.builder()
                .id(AccountId.from(source.id()))
                .name(source.name())
                .scoring(source.scoring())
                .build();
    }
}

# Domain layer 
In the domain layer there would be two main types: 
* Command - for executing concrete command. 
* Query - for reading information 

## Commands 

@Component
@RequiredArgsConstructor
public class CreateAccountCommand implements Command<Account, Result<Error, Account>> {

    private final AccountRepository accountRepository;

    public Result<Error, Account> execute(Account account) {
        return accountRepository.findById(account.getId())
                .map(a -> Result.fail(a, Error.ACCOUNT_ALREADY_EXIST))
                .orElseGet(() -> save(account));
    }

    private Result<Error, Account> save(Account account) {
        try {
            return Result.ok(accountRepository.save(account));
        } catch (DataIntegrityViolationException e) {
            return Result.fail(account, Error.ACCOUNT_ALREADY_EXIST);
        }
    }

    public enum Error {
        ACCOUNT_ALREADY_EXIST
    }

}

## Query 
HERE 



poprawic @Transactional 
uzywaj Grapth cos tam 
import javax.persistence.NamedEntityGraphs; @NamedEntityGraphs



