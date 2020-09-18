---
layout: post
title: "Functional tests with testcontainers.org library and Spring Boot"
categories: tests
---

In this article, you will find information on:
* How to write functional tests with testcontainers.org library. 
Before you read this article, I highly recommend first reading the one about [integration tests](2018-12-01-integration-tests-with-testcontainers.md)
You can follow a working example of the idea [here] (https://github.com/marekhudyma/application-style).

# Definition

According to the Wikipedia definition of [functional test](https://en.wikipedia.org/wiki/Functional_testing)

> Functional testing is a quality assurance (QA) process and a type of black-box testing that bases its test cases on the specifications of the software component under test. Functions are tested by feeding them input and examining the output, and internal program structure is rarely considered (unlike white-box testing).
> Functional testing is conducted to evaluate the compliance of a system or component with specified functional requirements. Functional testing usually describes what the system does

# Concept 
The main idea of functional tests is to make it independent from actual implementation. To achieve this: 
* **pack and run your service into docker container**, 
* **run all its dependencies** like: database, queues, streams, **as separate docker container**. 
* **make your testing code independent from implementation**. I do it by having multi-module project with `service` and `functional-tests`.

The structure of invocation can look like below.
<figure>
  <img src="/assets/2020-04-01-functional-tests-with-testcontainers/functional_tests.png" alt="Functional Tests"> 
  <figcaption>Functional Tests</figcaption>
</figure>
To summarize the main concept: Your entire production code needs to be packed and run as a docker image. If your service needs to communicate to the database, run the database as a docker image as well. 
Your `functional tests` will test your code run as docker image, so your testing code does not have any connection to production code. 

You also need to remember that a proper pyramid of tests is (from the biggest amount to the smallest amount): 
* unit tests 
* component tests 
* integration tests
* functional tests 
* system tests 

It is very nice to have functional tests, but it cannot dominate your testing structure.

## Packing into docker container 
Packing into docker image is pretty simple. In the root of your application, just define `Dockerfile` like: 

```
FROM openjdk:14-alpine
COPY service/target/application-style-exec.jar application-style.jar
EXPOSE 8080
ENTRYPOINT java --enable-preview ${ADDITIONAL_JAVA_OPTIONS} -jar application-style.jar
```

## Code separation 
I recommend organizing code into a multi-module project with two modules: `service` and `functional-tests`. The `functiona-tests` module cannot have any dependency to `service`. 
```
.
├── service
│   └── pom.xml
├── functional-tests
│   └── pom.xml
├── Dockerfile
└── pom.xml
```

## Rules 
Because we don't have access to production code, we cannot use any `DTO` objects, `database repositories`, etc. 

* We should operate on the simplest possible interfaces. For example, if we call `REST` endpoint, send plain `JSON` and read `JSON`. Don't create any internal `DTOs`.
* I recommend using only official interfaces to create resources. We could create the entity directly inside the database and inside test to just retrieve it. In my opinion, it would not be a black-box test then. If service changes storage in the future, we would need to change our black-box tests.

# AbstractFunctionalTests
All functional tests extend AbstractFunctionalTest where all needed docker images are run.
In our example, I will run my microservice which is connected to the database.

```java
public class AbstractFunctionalTest {

  private static final int HTTP_PORT = 8080;
  private static final int DEBUG_PORT = 5005;
  private static final Logger LOGGER = LoggerFactory.getLogger("Docker-Container");
  private static final Network network = Network.newNetwork();

  public static final PostgreSQLContainer postgreSQLContainer = 
    (PostgreSQLContainer) new PostgreSQLContainer("postgres:12.4")
      .withUsername("username")
      .withPassword("password")
      .withDatabaseName("application-style")
      .withNetwork(network)
      .withNetworkAliases("postgres");

  private static final GenericContainer<?> backendContainer;

  static {
    postgreSQLContainer.start();
    backendContainer = ofNullable(System.getenv("CONTAINER_VERSION"))
      .map(version -> new ServiceContainer("docker-repository/application-style", version))
      .orElseGet(() -> new ServiceContainer(".", Paths.get("../")))
      .withExposedPorts(HTTP_PORT, DEBUG_PORT)
      .withFixedExposedPort(DEBUG_PORT, DEBUG_PORT)
      .withEnv("SPRING_PROFILES_ACTIVE", "functional")
      .withEnv("ADDITIONAL_JAVA_OPTIONS", 
        "-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=0.0.0.0:" + DEBUG_PORT)
      .withNetwork(network)
      .withCreateContainerCmdModifier(cmd -> cmd.withName("application-style"))
      .withLogConsumer(new Slf4jLogConsumer(LOGGER).withPrefix("Service"))
      .waitingFor(Wait.forHttp("/actuator/health").forPort(HTTP_PORT)
        .withStartupTimeout(Duration.ofMinutes(2)));

      backendContainer.start();

      // make sure that containers will be stop in fast way (Ryuk can be slow)
      Runtime.getRuntime().addShutdownHook(new Thread(() -> {
        LOGGER.info("DockerContainers stop");
        backendContainer.stop();
        postgreSQLContainer.stop();
      }));
    }
}
```

# Debugging
Writing functional tests can be time consuming and difficult. Sometimes it is necessary to debug your code. Code is run as docker image. How do you debug it?
You can do it with `remote debugger`. First, you need to run your code with enabled remote debugger by passing options:
```java
.withEnv("ADDITIONAL_JAVA_OPTIONS", 
 "-agentlib:jdwp=transport=dt_socket,server=y,suspend=y,address=0.0.0.0:" 
 + DEBUG_PORT)
```
In the code above, flag `suspend=y` tells JVM to wait for remote debugger to connect.
We also need to specify which port is exposed by container and which ports have fixed ports. If your CI/CD tool shares a machine between multiple builds, you can have a port conflict, so enable it locally only. 

```java
.withExposedPorts(HTTP_PORT, DEBUG_PORT)
.withFixedExposedPort(DEBUG_PORT, DEBUG_PORT)
```

The method `withFixedExposedPort` is protected  in GenericContainer (in TestContainer) class. That's why you need to create a new class and expose it as a public. 
```java
public class ServiceContainer extends GenericContainer<ServiceContainer> {
  public ServiceContainer withFixedExposedPort(int hostPort, int containerPort) {
    super.addFixedExposedPort(hostPort, containerPort, InternetProtocol.TCP);
    return this;
  }
}
```

When you run such functional test, your JVM waits for the remote debugger and connects to it. 
Inside IntelliJ add project `Remote`, define the same port as in the test (usually `5005`), select use module classpath: your sources and click debug.
    
# Logging 
It is critical to add logging to a service. Without it, you are completely blind if there is any error. Don't forget to add logger. 

```
  .withLogConsumer(new Slf4jLogConsumer(LOGGER).withPrefix("Service"))
```

# Stopping images 
One of the biggest advantage of TestContainers library is the fact that there is a `Ryuk` container which stops all other containers when an initial JVM process dies. 
It protects us from unwanted zombie containers (and networks, volumes) in the system. But if you run docker images from multiple maven modules, `Ryuk` image can be too slow and the build can crash.
That's why I additionally specify `shutdownHook`, which stops all docker images when tests are over.

# Example of a functional test

The example of functional tests can look like below. The testing method uses many helper methods to simplify the test. In my opinion this is a key factor: create helper methods to make the code readable.

```java
public class AccountFunctionalTest extends AbstractFunctionalTest {
  
  @Test
  void shouldUpdateAccount() throws JSONException {
    // given
    createAccount("00000000-0000-0000-0000-000000000001");

    // when
    ResponseEntity<String> response = getTestRestTemplate()
      .exchange("/accounts/00000000-0000-0000-0000-000000000001",
      HttpMethod.PATCH,
      new HttpEntity<>(readFromResources("account_functional_test/patch_account_dto.json"), 
        getPatchHeaders(etag)),
      String.class);

    // then
    assertThat(response.getStatusCodeValue()).isEqualTo(HttpStatus.NO_CONTENT.value());
    var actual = getAccount("00000000-0000-0000-0000-000000000001");
    var expected = readFromResources("account_functional_test/get_account_dto.json");
    JSONAssert.assertEquals(expected, actual, JSONCompareMode.LENIENT);
  }
  
  private void createAccount() {
    var json = readFromResources("account_functional_test/create_account_dto.json");
    ResponseEntity<String> response = getTestRestTemplate().exchange("/accounts",
            HttpMethod.POST,
            new HttpEntity<>(json, getPostHeaders()),
            String.class);
    assertThat(response.getStatusCodeValue()).isEqualTo(HttpStatus.CREATED.value());
  }

  private String getEtag(String id) {
    ResponseEntity<String> response = getTestRestTemplate()
      .getForEntity("/accounts/{id}", String.class, id);
    return response.getHeaders().getETag();
  }

  private String getAccount(String id) {
    ResponseEntity<String> response = getTestRestTemplate()
      .getForEntity("/accounts/{id}", String.class, id);
    return response.getBody();
  }

  private HttpHeaders getPostHeaders() {
    HttpHeaders headers = new HttpHeaders();
    headers.setContentType(MediaType.APPLICATION_JSON);
    return headers;
  }

  private HttpHeaders getPatchHeaders(String etag) {
    HttpHeaders headers = new HttpHeaders();
    headers.setContentType(new MediaType("application", "merge-patch+json"));
    headers.add(HttpHeaders.ETAG, etag);
    return headers;
  }
}
```

# Difference between Integration and Functional Tests
If you read the previous article about [integration tests](2018-12-01-integration-tests-with-testcontainers.md), you may ask: What is actual difference?
These two types of tests may be pretty similar, I will highlight the most important differences:
* In FunctionalTests, you are not allowed to use any production code.
* All actions need to be done by public interfaces of your system (e.g. nothing is created directly inside the database),
* Inside functional tests should test happy paths, not corner cases. In contrast, inside integration tests can simulate e.g. timeouts, bad responses. etc. 

# Advantages of functional tests
For me, the biggest advantages of functional tests are:
* We are able to test the service as black-box, meaning that that when you have a good functional tests coverage, you are able to make a deep refactoring without changing functional tests. 
* It gives developers pretty big confidence that the code does what it should do. 

# Disadvantages of Functional Tests
* I need to admit that writing functional tests can be time consuming. Especially when something doesn't work as expected, debugging became much harder.  
* Because functional tests are running service and dependencies (like database, queues) as docker images, we need to run it at least once. Usually it slow. 
* In an ideal world, we should run new containers for each test, but it would be way too slow. So, we need to run it once for all tests. If functional tests are written in a bad way, they can make tests interfere each other. 
It is critical that tests use different object identifiers and there is clean state after the test. 

# Summary 
I find functional tests to be a interesting concept. TestContainers library make it possible to use this concept inside Java world. 
It can be pretty expensive to implement it, but it also gives you big confidence that a system still works during deep refactoring.
