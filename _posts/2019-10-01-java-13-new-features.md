---
layout: post
title: "Java 13 new features"
featured: false
author: marek
categories: Java
type: post
image: '/assets/2019-10-01-java-13-new-features/java13.png'
comments: false
---

<figure>
  <img src="/assets/2019-10-01-java-13-new-features/java13.png" alt="Java 13" />
</figure>

Java 13 was released on 17 September 2019. Let's see what new features it provided.

# Syntax features

##[JEP 355: Text Blocks (Preview)](https://openjdk.java.net/jeps/355)
The preview feature has multi-line Strings. In the past storing `JSON` as String required a lot of escaping like:
```java
String JSON = "{\"version\": 13,\"language\": \"java\"}";
```
With this feature, we can use multi-line Strings without needing to escape double quotes or to add a carriage return:
```java
String JSON2 = """
{
  "version": 13,
  "language": "java"
}
""";
```

## [JEP 354: Switch Expressions (Second Preview)](https://openjdk.java.net/jeps/354)

The switch feature was introduced as preview in Java 12, via [JEP 325: Switch Expressions (Preview)](https://openjdk.java.net/jeps/325).
Feedback was sought initially on the design of the feature, and later on the experience of using switch expressions and the enhanced switch statement.

Switch was extended by `yield` statement.
The `yield` statement `exits` the switch and `returns` the result of the current branch, similar to a return.

```java
int cost = switch (day) {
  case MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY  -> {
    yield 1;
  }
  case SATURDAY, SUNDAY -> {
    yield 2;
  }
};
```

## java.lang.String
String has new methods:

* `stripIndent()` – mimics the compiler to remove incidental white space
* `translateEscapes()` – translates escape sequences such as “\\t” to “\t”
* `formatted()` – works the same as String::format, but for text blocks


## [JEP 350: Dynamic CDS Archives](https://openjdk.java.net/jeps/350)

Extend application class-data sharing to allow the dynamic archiving of classes at the end of Java application execution.
The archived classes will include all loaded application classes and library classes that are not present in the default, base-layer CDS archive.

To use it, you need to add JVM option
```java
-XX:ArchiveClassesAtExit=<archive filename>
```
To include archive file during the start, use:
```java
-XX:SharedArchiveFile
```

## [JEP 351: ZGC: Uncommit Unused Memory (Experimental)](https://openjdk.java.net/jeps/351)

Enhance ZGC to return unused heap memory to the operating system.

## [JEP 353: Reimplement the Legacy Socket API](https://openjdk.java.net/jeps/353)

Replace the underlying implementation used by the `java.net.Socket` and `java.net.ServerSocket` APIs with a simpler and more modern implementation which is easy to maintain and debug.

# Summary 
Java 13 was, for me, just another iteration of improvements without huge changes in stable features. 