---
layout: post
title: "Java 10 new features"
featured: false
author: marek
categories: Java
type: post
image: '/assets/2019-02-01-java-10-new-features/Java10.jpg'
comments: false
---


<figure>
  <img src="/assets/2019-02-01-java-10-new-features/java10ready.png" alt="Java 10 ready">
</figure>
In this blog article, we will follow the new features of Java 10, released on 20 March 2018.

# Java 10 syntax sugar

## [JEP 286: Local-Variable Type Inference](http://openjdk.java.net/jeps/286)

A keyword `var` was introduced - type inference of local variables with initializers. A feature inspired by other languages allows skipping the type of variable during initialization.
```java
String message = "Before Java 10";
var message = "From Java 10";
```

Worth mentioning that using var:
* initialization is required
* illegal use: `var message;` or `var message = null;`
* var keyword can only be used to declare a local variable (e.g. inside method)
* you cannot use var to declare member variables inside the class, formal parameters, or to return the type of methods.

## Unmodifiable Collections

Java classes:
* java.util.List
* java.util.Map
* java.util.Set
got a new `static method copyOf(Collection)` that returns an unmodifiable copy of the collection, for example:

```java
List<Integer> unmodifiableList = List.copyOf(mutableList);
```

## stream - toUnmodifiable()

Java streams got new collectors that collect unmodifiable List, Map, Set.
example:
```java
    List<Integer> unmodifiableList = myList.stream()
    ...
      .collect(Collectors.toUnmodifiableList());
```

## Optional.orElseThrow()
By using `Optional.orElseThrow` method and the value is not present, an exception `NoSuchElementException` will be thrown. 
It inlines to the existing `Optional.orElseThrow(Supplier<? extends X> exceptionSupplier)` implementation used by consumers as an explicit alternative.
Example: 
```java
var value = myOptional.orElseThrow();
```

Old equivalent:
```java
var value = myOptional.orElseThrow(NoSuchElementException::new);
```

# Performance optimizations

## [JEP 307: Parallel Full GC for G1](http://openjdk.java.net/jeps/307)

The `G1 garbage collector` is the default one since JDK 9. However, the `full GC for G1` used a `single-threaded mark-sweep-compact` algorithm.
This has been changed to the `parallel mark-sweep-compact` algorithm in Java 10 effectively reducing the stop-the-world time during full GC.

## [JEP 310: Application Class-Data Sharing](https://openjdk.java.net/jeps/310)

In JDK 5, the feature `Class-Data Sharing (CDS)` has been introduced. It allows a set of classes to be pre-processed into a shared archive file that can then be memory-mapped at runtime to reduce startup time which can also reduce dynamic memory footprint when multiple JVMs share the same archive file.

In JDK 10, the feature `Application Class Data Sharing (AppCDS)` has been introduced.  The idea behind AppCDS is to `share` once loaded classes between JVM instances on the same host.

What is the difference?:
* Different JVM instances on the same host often load the same classes,
* CDS - store `JDK classes` into an `archive` and share it between JVM instances,
* AppCDS - do the same with `application classes` and `third party library classes`.

`AppCDS` can improve the footprint and runtime of your application.

Useful links: [link1](https://www.linkedin.com/pulse/java-10-application-class-data-sharing-abhi-kerni/), [link2](https://www.baeldung.com/java-10-performance-improvements#application-class-data-sharing), [link3](https://medium.com/@toparvion/appcds-for-spring-boot-applications-first-contact-6216db6a4194)


## [JEP 317: Experimental Java-Based JIT Compiler](http://openjdk.java.net/jeps/317)

JDK 10 enables the [Graal](https://github.com/oracle/graal/blob/master/compiler/README.md) compiler, to be used as an experimental JIT compiler on the Linux/x64 platform.
To enable Graal as the JIT compiler, use the following options on the java command line:
```java
-XX:+UnlockExperimentalVMOptions -XX:+UseJVMCICompiler
```
Warning: it is still experimental and nobody guarantees performance improvement.

# Container Awareness
JVMs are now aware of being run in a Docker container and will extract container-specific configuration instead of querying the operating system itself – it applies to data like the number of CPUs and total memory that have been allocated to the container.

You can reliably set a number of processors:
```java
-XX:ActiveProcessorCount=count
```
and better control memory settings:
```java
-XX:InitialRAMPercentage
-XX:MaxRAMPercentage
-XX:MinRAMPercentage
```

Read more [here](https://www.docker.com/blog/improved-docker-container-integration-with-java-10/) and [here](https://medium.com/@jnsrikanth/docker-support-in-java-10-fbff28a31827).

This feature has been backported to `Java 8` as well, read [here](https://merikan.com/2019/04/jvm-in-a-container/).

# [JEP 319: Root Certificates](http://openjdk.java.net/jeps/319)

With Java 10, Oracle has open-sourced the root certificates in Oracle's Java SE Root CA program in order to make OpenJDK builds more attractive to developers and to reduce the differences between those builds and Oracle JDK builds.

Read more [here](https://dzone.com/articles/openjdk-10-now-includes-root-ca-certificates)

# Cleanups
Tools removed:
* `javah` (command generates C header and source files that are needed to implement native methods) - use `javac -h` instead,
* `policytool` (UI based tool for policy file management) - use text editor,
* `java -Xprofoption` (option for profiling) - use `jmap`.

Deprecated:
* `java.security.acl package` - use `java.security.Policy` instead,
* `java.security.{Certificate,Identity,IdentityScope,Signer}`

# Java releases

From Java 10, it will be released every 6 months, in March and September - these are called feature releases.
In the moment of release of a new feature release of Java, the older feature release is no longer supported.
Long-term support release will be marked as LTS.

You can check General Availability by executing the command.
```
java -version
openjdk version "10" 2018-03-20
```

# Summary

I described the most important features of Java 10. It was not a significant release, but it surprised me how many new and small improvements were added to this release.

For more detailed information I can recommend the official [Java 10 release page](http://openjdk.java.net/projects/jdk/10/).