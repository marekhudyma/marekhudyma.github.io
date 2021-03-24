---
layout: post
title: "Java 11 new features"
categories: Java
---

<figure>
  <img src="/assets/2019-03-01-java-11-new-features/java11.png" alt="Java 11" />
</figure>

In this article we will follow the most important new features of Java 11. 
Java 11 was released in September 2018 as Long Term Support version. 
Oracle JDK is the first version that cannot be used for free commercially.

# Syntax features


## String methods

New methods were added to String:

* `isBlank` - true if the string is empty or contains only white space codepoints, otherwise false,
* `lines` - returns a stream of lines extracted from this string, separated by line terminators,
* `strip` - returns a string whose value is this string, with all leading and trailing white space removed,
* `stripLeading` - returns a string whose value is this string, with all leading white space removed,
* `stripTrailing` - returns a string whose value is this string, with all trailing white space removed,
* `repeat` - returns a string whose value is the concatenation of this string repeated count times.

## [JEP 327: Unicode 10](http://openjdk.java.net/jeps/327)

Java 11 supports the latest version of `Unicode 10`. Now you can use `emojis` in your code!
```java
System.out.println("😃"); 
System.out.println("\uD83D\uDE80"); //An alternative way of displaying emojis.
```

## New File Methods

Class java.nio.file.Files has two methods:

* `Files.readString(Path path)`,
* `Files.writeString(Path path)`

## Collection.toArray

An interface Collection has a new method `toArray(IntFunction)`. 
This makes it easier to create an array of the right type from a collection:
```java
List<Integer> myList = List.of(1, 2, 3);
Integer[] array = myList.toArray(Integer[]::new);
```

## Stream Predicate NOT

A new stream predicate not has been introduced, example:
```java
List<String> listWithoutEmpty = listOfString.stream() 
    .filter(not(String::isEmpty)) 
    .collect(Collectors.toList());
```

## [JEP 323: Local-Variable Syntax for Lambda Parameters](https://openjdk.java.net/jeps/323)

Java stream got another nice feature: the possibility of using var in lambda functions, example.
It is important because we can now also add an annotation to the variable without specifying variable type.

```java
import org.jetbrains.annotations.NotNull;
..
List<String> fileNames = List.of("").stream() 
  .map((@NotNull var text) -> text + ".txt") 
  .collect(Collectors.toList());
```

## [JEP 321: HTTP Client](https://openjdk.java.net/jeps/321)

Standardized the incubated HTTP Client API introduced in JDK 9, via [JEP 110](https://openjdk.java.net/jeps/110) and updated in JDK 10. 
From Java 11 it became a standard feature. Worth mentioning that it can execute asynchronous calls. Example of usage:
```java
HttpRequest request = HttpRequest.newBuilder() 
    .uri(URI.create("https://www.zalando.de")) 
    .GET() // default
    .build();

var response = httpClient.send(request, BodyHandlers.ofString());
System.out.println("status code: " + response.statusCode());
System.out.println("headers: " + response.headers());
System.out.println("body: " + response.body());
```

## [JEP 181: Nest-Based Access Control](https://openjdk.java.net/jeps/181)
`java.lang.Class` got a few new methods: `isNestmateOf`, `getNestHost`, `getNestMembers`. 
Let’s follow the example First we have Outer and Inner classes.

```java
public class Outer { 
  class Inner { 
    
  }
}
```
Here we can see the behaviour.
```java
assertThat(Outer.class.isNestmateOf(Inner.class)).isTrue();
assertThat(Inner.class.getNestHost()).isEqualTo(Outer.class);
assertThat(Outer.class.getNestMembers()) .isEqualTo(new Class[] {Outer.class, Inner.class});
```
It simplifies the access model between Outer and Inner classes, even for refection (that was not possible before).

## [JEP 330: Launch Single-File Source-Code Programs](https://openjdk.java.net/jeps/330)

You do not need to compile single file programs. Instead of compiling and running:
```java
$ javac HelloWorld.java
$ java HelloWorld
Hello World!
```

You can directly run it with java command:
```java
$ java HelloWorld.java
Hello World!
```

# Performance

[JEP 315: Improve Aarch64 Intrinsics](https://openjdk.java.net/jeps/315)

Optimized:
* `String`,
* `array intrinsics`,
* maths: `Math.sin()`, `Math.cos()` and `Match.log()` on [Arm64 architecture](https://en.wikipedia.org/wiki/ARM_architecture)

[JEP 318: Epsilon: A No-Op Garbage Collector (Experimental)](https://openjdk.java.net/jeps/318)

A new Epsilon Garbage Collector was added to Java 11. It allocated memory but does not actually collect any garbage. It can be useful in case of:

* Performance testing
* Extremely short-lived jobs. It stops virtual machine when memory is exhausted.

In order to enable it, use flag:
```java
-XX:+UnlockExperimentalVMOptions -XX:+UseEpsilonGC
```

## Flight Recorder

`Java Flight Recorder (JFR)` was a commercial product in Oracle JDK, now it is open source in OpenJDK 11. 
JFR is a profiling tool for Java application. You can run profiling for a given application by running command:dumps the recorded data into a .jfr file.

```java
java -XX:StartFlightRecording=duration=60s,settings=profile,filename=helloWorld.jfr HelloWorld
```

As a result, it will produce the file: `helloWorld.jfr`, which you can analyze and visualize with `Java Mission Control (JMC)`.

## [Cleanups - JEP 320: Remove the Java EE and CORBA Modules](https://openjdk.java.net/jeps/320)

Removed packages:

* `java.xml.ws` (JAX-WS)
* `java.xml.bind` (JAXB)
* `java.activation` (JAF)
* `java.xml.ws.annotation` (Common Annotations)
* `java.corba` (CORBA)
* `java.transaction` (JTA)
* `java.se.ee` (Aggregator module for the six modules above)

Removed Tools:

* wsgen and wsimport (from `jdk.xml.ws`)
* schemagen and xjc (from `jdk.xml.bind`)
* `idlj`, `orbd`, `servertool`, and `tnamesrv` (from `java.corba`)

Deprecated Modules:

* `Nashorn JavaScript engine`, including the JJS tool
* `Pack200 compression schemes` for JAR files

## [JEP 332: Transport Layer Security (TLS) 1.3](http://openjdk.java.net/jeps/332)

The JDK 11 release includes an implementation of the Transport Layer Security (TLS) 1.3 specification [(RFC 8446)](https://tools.ietf.org/html/rfc8446).

# Summary

In this article we outlined the changes in Java 11. 
Personally, I find syntax sugar added to String very helpful. I think I will also use emojis more often. 😃
