---
layout: post
title: "Java 14 new features"
categories: Java
---

<figure>
  <img src="/assets/2020-11-01-java-14-new-features/java.png" alt="Java 14" />
</figure>

Java 14 was released on March 17, 2020. Let's see its new features. 

# Syntax features

## [JEP 361: Switch Expressions](https://openjdk.java.net/jeps/361)
New switch syntax was proposed in Java 12 as [JEP 325: Switch Expressions (Preview)](https://openjdk.java.net/jeps/325)
later it was slightly changed in [JEP 354: Switch Expressions (Second Preview)](https://openjdk.java.net/jeps/354)

Finally, it became a standard feature, example of syntax: 

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

## [JEP 368: Text Blocks (Second Preview)](https://openjdk.java.net/jeps/368)

This feature was proposed in `Java 13`, but was not included into standard. It got two new escape sequences:
* `\` - to indicate a new line, 
* `\s` - to indicate a `single space`.

String can be written in a way:
```java
String JSON2 = """
{ \
  "version": 13, \
  "language": "java" \
} \
""";
```

## [JEP 305: Pattern Matching for instanceof (Preview)](https://openjdk.java.net/jeps/305)

A new instanceof has been introduced to Java 14. The syntax eliminates boilerplate code:
```java
if (obj instanceof String s) { 
  // you can use variable s here
}
```

## [JEP 359: Records (Preview)](https://openjdk.java.net/jeps/359)

Records were introduced as `preview` feature. It is immutable class, example: 

```java
public record User(int id, String name) { };
```

`Record` generates for us: 
* `private final` fields, 
* `getters`,
* `constructor` with all fields, 
* `equals` method, 
* `hashCode` method,
* `toString` method.

By using `record` you can still: 
* redefine generated constructor, e.g. by adding validation,
* add a different constructor,
* add methods,
* implement interfaces.

By using `record` you `can't`:
* extend a class nor can it be extended by another class,
* be abstract,

`Compact constructor` is a syntax that allows adding of some common logic, e.g. validation when `canonical constructor` is called.
```java
public record User(int id, String name) {
  public User {
    if(id< 100) {
      throw new java.lang.IllegalArgumentException(String.format("Invalid id: %d",id));
    }
  }
}
```

Explicit declaration of field accessor method:
```java
public String name() {
  System.out.println(name);
  return name;
}
```

## [JEP 358: Helpful NullPointerExceptions](https://openjdk.java.net/jeps/358)

In Java 14 were introduced a more descriptive NullPointerExceptions, example:
```java
User user = null;
int id = user.getId();
```
Exception:
```java
java.lang.NullPointerException: Cannot invoke "com.java.Java12$User.getId()"
    because "user" is null
```

## [JEP 370: Foreign-Memory Access API (Incubator)](https://openjdk.java.net/jeps/370)
Introduce an API to allow Java programs to safely and efficiently access foreign memory outside of the Java heap.

## [JEP 343: Packaging Tool (Incubator)](https://openjdk.java.net/jeps/343)

From `Java 11` JavaFX did not belong to `JDK`. Also tool `javapackager` was no longer available.
In `Java 14` a new tool was introduced `jpackage`. It packages whole application with JDK into executable file. 

## [JEP 365: ZGC on Windows (Experimental)](https://openjdk.java.net/jeps/365), [JEP 364: ZGC on macOS (Experimental)](https://openjdk.java.net/jeps/364)
`ZGC` was introduced in `Java 13`. Java 14 has ported its support to Windows and macOS as well.

## [JEP 345: NUMA-Aware Memory Allocation for G1](https://openjdk.java.net/jeps/345)

Improve G1 performance on large machines by implementing NUMA-aware memory allocation.

## [JEP 349: JFR Event Streaming](https://openjdk.java.net/jeps/349)

`JDK Flight Recorder` was introduced in `Java 11`, now data are exposed for continuous monitoring.

# Cleanings

Here I will only mention the most important cleanings made in the JDK. 
* `Concurrent Mark Sweep (CMS) Garbage Collector` ([JEP 363](https://openjdk.java.net/jeps/363)) – has been deprecated by `Java 9`, now has been removed.
The recommendation is to use G1. Also there are other alternatives, e.g. `ZGC`, `Shenandoah`.
* `Pack200 Tools and API` ([JEP 367](https://openjdk.java.net/jeps/367)) – was deprecated by `Java 11`, now has been removed. 

# Summary
Java 14 brought many preview features as standard. For me, it will always stay in memory as the version of Java which proposed experimental version of `records`.