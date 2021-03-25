---
layout: post
title: "Java 16 new features"
categories: Java
---

<figure>
  <img src="/assets/2021-04-01-java-16-new-features/java16.jpg" alt="Java 16" />
</figure>

Java 16 was released on March 16, 2020. Let's see its new features.

# Syntax features

## [JEP 394: Pattern Matching for instanceof](https://openjdk.java.net/jeps/394)

`Pattern Matching for instanceof` was introduced in `Java 14` and in `Java 15`.
In `Java 16`, it became a production feature. It is possible to write statements like:

```java
if (obj instanceof String s && s.length() > 5) {
  flag = s.contains("jdk");
}
```

## [JEP 395: Records](https://openjdk.java.net/jeps/395)

`Records` were introduced in `Java 14` and again in `Java 15`. In `Java 16`, it became a production feature.
According to `JEP` documentation, the mail purpose of records is not to reduce boiler plate code. 
The main goal is to provide truly immutable objects. Example of usage:

```java
public record User(int id, String name) { };
```

Also `local record` became a production feature.
```java
List<Merchant> findTopMerchants(List<Merchant> merchants, int month) {
// Local record
record MerchantSales(Merchant merchant, double sales) {}
  return merchants.stream()
    .map(merchant -> new MerchantSales(merchant, computeSales(merchant, month)))
    .sorted((m1, m2) -> Double.compare(m2.sales(), m1.sales()))
    .map(MerchantSales::merchant)
    .collect(toList());
}
```

## [JEP 376: ZGC: Concurrent Thread-Stack Processing](https://openjdk.java.net/jeps/376)

This feature moves ZGC thread-stack processing from safepoints to a concurrent phase, allows sub-millisecond pauses inside GC safepoints, even on large heaps.
Removing the final source of latency in the ZGC garbage collector will greatly improve performance and efficiency of applications in this and subsquent releases.

## [JEP 387: Elastic Metaspace](https://openjdk.java.net/jeps/387)

This feature returns unused HotSpot VM class-metadata (i.e. metaspace) memory to the operating system more promptly, reducing metaspace footprint.

## [JEP 392: Packaging Tool](https://openjdk.java.net/jeps/392)

The `jpackage` tool was introduced as an incubating tool in `JDK 14` by `JEP 343`. It remained an incubating tool in `JDK 15`.
The main goal of `jpackage` is to create a packaging tool, based on the legacy JavaFX javapackager tool, that:

* Supports native packaging formats to give end users a natural installation experience. These formats include `msi`, `exe`, `dmg`.

## [JEP 396: Strongly Encapsulate JDK Internals by Default](https://openjdk.java.net/jeps/396)

Strongly encapsulate all internal elements of the JDK by default, except for critical internal APIs such as sun.misc.Unsafe. Allow end users to choose the relaxed strong encapsulation that has been the default since JDK 9.
Code successfully compiled with earlier releases that accesses internal APIs of the JDK may no longer work by default.

## [JEP 338: Vector API (Incubator)](https://openjdk.java.net/jeps/338)

This offers an initial iteration of an incubator module, to express vector computations that reliably compile at runtime to optimal vector hardware instructions and thus achieve superior performance to equivalent scalar computations.

## [JEP 389: Foreign Linker API (Incubator)](https://openjdk.java.net/jeps/389)

This incubator API offers statically-typed, pure-Java access to native code

## [JEP 393: Foreign-Memory Access API (Third Incubator)](https://openjdk.java.net/jeps/393)

First introduced as an incubator API in `Java 14` and again in `Java 15`, this API allows Java programs to safely and efficiently operate on various kinds of foreign memory (e.g., native memory, persistent memory, managed heap memory, etc.). It also provides the foundation for the Foreign Linker API.

## [JEP 397: Sealed Classes (Second Preview)](https://openjdk.java.net/jeps/397)

Sealed Classes were proposed and delivered as a preview feature in JDK 15. It stayed in preview.

## [JEP 347: Enable C++14 Language Features](https://openjdk.java.net/jeps/347)

This allows the use of C++14 language features in `JDK C++ source code` and gives specific guidance about which of those features may be used in HotSpot code.

## [JEP 357: Migrate from Mercurial to Git](https://openjdk.java.net/jeps/357) && [JEP 369: Migrate to GitHub](https://openjdk.java.net/jeps/369)

These JEPs migrate the OpenJDK Community's source code repositories from Mercurial to Git and host them on GitHub for JDK 11 and later.

# Summary 
`Java 16` release mostly accepted production features. Extended previews I expect to see merged in `Java 17` that will be `Long Term Support`.