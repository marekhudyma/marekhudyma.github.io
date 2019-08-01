---
layout: post
title: "Integration tests with testcontainers.org library and Spring Boot"
categories: tests
---

In this article, you will find information on:
* how to write integration tests with testcontainers.org library. 

# Presentation
<iframe src="//www.slideshare.net/slideshow/embed_code/key/ImUcqZ9yyHPg1H" width="595" height="485" frameborder="0" marginwidth="0" marginheight="0" scrolling="no" style="border:1px solid #CCC; border-width:1px; margin-bottom:5px; max-width: 100%;" allowfullscreen> </iframe> <div style="margin-bottom:5px"> <strong> <a href="//www.slideshare.net/MarekHudyma/integration-tests-with-testcointainers-library" title="Integration tests with testcointainers library. " target="_blank">Integration tests with testcointainers library. </a> </strong> from <strong><a href="https://www.slideshare.net/MarekHudyma" target="_blank">Marek Hudyma</a></strong> </div>
# Integration testing

Integration testing is the phase in software testing in which individual software modules are combined and tested as a group.
It occurs after unit testing.

# Introduction to the 
# FIRST - Principles for Writing Good Tests
All tests (including integration tests) should follow principles defined as ```FIRST```. 
Acronym ```FIRST``` stand for below test features:

* **[F]**ast - A test should not take more than a second to finish the execution
* **[I]**solated/Independent - No order-of-run dependency. They should pass or fail the same way in suite or when run individually. Do not depend on any external resources.
* **[R]**epeatable - A test method should NOT depend on any data in the environment/instance in which it is running.
* **[S]**elf-Validating - No manual inspection required to check whether the test has passed or failed.
* **[T]**horough - Should cover every use case scenario and NOT just aim for 100% coverage

Pyramid of testing
<figure>
  <img src="/assets/2018-12-01-integration-tests-with-testcontainers/pyramid_of_testing.png" alt="Pyramid of testing"> 
  <figcaption>Pyramid of testing</figcaption>
</figure>
As you can see in pyramid of testing, number of test should be average - more then manual and system tests, more than component and unit tests.

# Concept of integration tests with testcontainers.org library
testcontainers.org  is a Java library that allow to run docker images and control it from Java code.  (I will not cover topic what is Docker, if you need more information <a href="https://en.wikipedia.org/wiki/Docker_(software)">read more</a> about it.)

The main concept of the proposal of integration test is: 
* Run your application 
* Run ```external components``` as real docker containers. Here is important to understand what do I mean by ```external components```. It can be: 
    * database storage - for example run real PostgreSQL as docker image, 
    * Redis - run real Redis as docker image, 
    * RabbitMQ
    * AWS components like S3, Kinesis, DynamoDB and other you can emulate by ```localstack``` 
    
**Don't run another microservice as docker image**. If you communicate with other microservice via HTTP, mock requests by ```mockserver``` run as docker image. 

<figure>
  <img src="/assets/2018-12-01-integration-tests-with-testcontainers/concept.jpg" alt="Concept"> 
  <figcaption>Your service comunicates with external components run as docker image. </figcaption>
</figure>

## Advantages 

* You run test against real components, for example H2 database doesn't support Postgres/MySQL specific functionality. 
* You can run your tests offline - no Internet connection needed. It is advantage for people who are traveling or if you have slow Internet connection. 
* You can mock AWS services by ```localstack```. It will simplify administrative actions, cut costs and make your build offline.
* You can test cornercases like: 
    * simulate timeout from external service, 
    * simulate wrong http codes,
* All tests are written by developers in the same commit. 

## Disadvantages
* Continuous integration (eg Jenkins) machine need to be bigger (build uses more RAM and CPU).
* You need to run containers at least once - it consumes time and resources.

# How to run Integration Tests

To run PostgreSQL you can add it to your test:
```
protected static PostgreSQLContainer postgreSQLContainer = new PostgreSQLContainer("postgres:11.2")
        .withUsername("userName")
        .withPassword("password")
        .withDatabaseName("experimentDB");
```
Than run database migration scripts (eg. Flyway). Even empty test will validate if your migrations are executed properly.
As a good practice, you should remember about cleaning the state - delete inserted rows.

## Example of DB IntegrationTest
```
    @BeforeEach
    void setUp() throws Exception {
        name = UUID.randomUUID().toString();
    }
    @AfterEach
    void tearDown() throws Exception {
        accountRepository.findByName(name)
            .ifPresent(account -> accountRepository.delete(account));
    }
    @Test
    void shouldFindByName() throws Exception {
        Account account = Account.builder()
            .name(name).additionalInfo("additionalInfo").build();
        accountRepository.save(account);
    }
    Optional<Account> actual = accountRepository.findByName(name);
        assertThat(actual.get()).isEqualTo(account);
    }
```

## Testing queues
To run RabbitMQ add it to the test:
 ```
    private static GenericContainer rabbitMqContainer =
        new GenericContainer("rabbitmq:3.7.7").withExposedPorts(5672);
```
Testing queues is more tricky, because it is asynchronous. You don't know how much time you need to consume message after putting it to the queue. 

* If your production code receive the message, in the test put it in the queue. Then wait for consumption of the message and validate if changes made by consumption are what you expect. 
You can use <a href="https://github.com/awaitility/awaitility">Awaitility</a> library to validate results of test.
* If your production code sends a message, create a testConsumer in test scope and validate if you receive proper messages.


**Always clean the queue after the test.** Every send message should be consumed. If you don't do it, your build can became unpredictable. 


## Testing external HTTP calls REST / SOAP
To simulate interaction via HTTP use mockserver library (as docker image).

Testing external http calls - example:
 ```
mockServerContainer.getClient().reset(); //reset the mock server container
HttpRequest request = request("/api/entity/name.1").withMethod("GET");
    getMockServerContainer().getClient()
    .when(request)
    .respond(response()
    .withHeader(HttpHeaders.CONTENT_TYPE, MediaType.APPLICATION_JSON.toString())
    .withStatusCode(200)
    .withDelay(TimeUnit.MILLISECONDS, timeoutInMs + DELTA_IN_MS)
    .withBody(readFromResources("additionalInfo1.json")));
 ```
The big advantage of this way of testing is that you can verify that the call happened:
 ```
    getMockServerContainer().getClient().verify(request);
 ```
    
## AWS testing with localstack
You can mock interaction with AWS with localstack. Right now localstack supports:
* API Gateway
* Kinesis
* DynamoDB
* DynamoDB Streams
* Elasticsearch
* S3
* Firehose
* Lambda
* SNS
* SQS
* Redshift
* SES
* Route53
* CloudFormation
* CloudWatch
* SSM
* SecretsManager

# How to start working with testcointainers library

One way of creating tests is extending abstract test class like example below. 
It contains static references to docker containers. In `static` block, we start all images. 
```
@ExtendWith(SpringExtension.class)
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@ContextConfiguration(initializers = AbstractIntegrationTest.Initializer.class)
public abstract class AbstractIntegrationTest {
    protected static PostgreSQLContainer postgreSQLContainer = new PostgreSQLContainer("postgres:10.4")
        .withUsername("userName")
        .withPassword("password")
        .withDatabaseName("experimentDB");
    protected static MockServerContainer mockServerContainer = new MockServerContainer("5.4.1");
    protected static GenericContainer rabbitMqContainer = new GenericContainer("rabbitmq:3.7.7")     
        .withExposedPorts(5672);
        
    static {
        postgreSQLContainer.start();
        rabbitMqContainer.start();
        mockServerContainer.start();
    }
```
 
## Closing resources
You don’t need to release any resources, there is: [Runtime.getRuntime().addShutdownHook()](https://github.com/testcontainers/testcontainers-java/blob/master/core/src/main/java/org/testcontainers/utility/ResourceReaper.java#L343) that closes all docker images.
There is a [ryuk docker image](https://quay.io/repository/testcontainers/ryuk?tag=latest&tab=tags) that does it.

# Eureka
For service discovery I am using Eureka. At the beginning I was running Eureka as docker image in my integration tests.
To save resources on CI I decided to switched it off: 
```
eureka:
  client:
    enabled: false
```
Than I mocked the addresses of external services, by setting system properties:
```
new ImmutableMap.Builder<String, String>()
   .put("commonsscriptsync.ribbon.listOfServers", format("localhost:%d", mockserverPort))
   .build()
   .forEach(System::setProperty);
```

# Cloud Auth
My microservices are using Cloud Auth for authorization.
To save resources on CI I decided to switched it off. I mocked the Cloud Auth by MockServer - hardcoded JWT token - valid for everything and forever.
```
requestAuth = request()
                .withPath("/oauth/token")
                .withMethod("POST");
getMockServerContainerClient()
                .when(requestAuth)
                .respond(response()
                        .withHeader("Content-Type", "application/json;charset=utf-8")
                        .withBody(readFromResources("integration/microservices/cloud_auth_response.json")));
```

# Disadvantages of testcontainer library
TestContainers supports docker_compose file, but only version 2.0, while docker_compose current version is 3.6.

# Alternative solutions
I liked TestContainers library. Personally I find `http://testcompose.com` library more interesting.
Main advantage is that it simply runs docker_compose file and reduce boilerplate code.

# Code example
You can find examples of usages in my github project: `https://github.com/marekhudyma/testcontainers/`
