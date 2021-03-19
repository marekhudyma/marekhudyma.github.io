---
layout: post
title: "Java 12 new features"
categories: Java
---

<figure>
  <img src="/assets/2019-04-01-java-12-new-features/java12.png" alt="Java 12" />
</figure>

Java 12 was released on 19 March 2019. 
Some of the features are only available only after using flag: `-enable-preview` during compilation. 
They are unstable and can change in the future. Let’s check what the new features of Java 12 are.


# Syntax features

## String

This release is another one in the row which extended String class.

## String.indent(int n)

`indent(int n)` - this method adjusts the indentation of each line in the string based on the value of `n` and also normalizes line termination characters.
if (n > 0) { 
  then n spaces are inserted at the beginning of each line.
} else if(n < 0) { 
  then up to n white spaces are removed from the beginning of each line. If line doesn’t contain n lines, then all leading white spaces are removed.
} else if (n == 0) { 
  then String is not being changed.
}
```java
String str = "Hello\nWorld!";System.out.println(str.indent(3));
```
Output:
```java
Hello World!
```

## String.transform(Function<? super String, ? extends R> f)

This method allows us to call a function on the given string. The function should expect a single String argument and produce an `R result`.
```java
String s = "Hello,World";
List<String> list = s.transform(s1 -> { 
  return Arrays.asList(s1.split(",")); 
});
```

## nio.file.Files.mismatch(Path path, Path path2)

Utility Files gained a new method mismatch. Two files can have a mismatch in the following scenarios:

* If the bytes are not identical. In this case, the position of the first mismatching byte is returned.
* File sizes are not identical. In this case, the size of the smaller file is returned.
```java
long mismatch = Files.mismatch(filePath1, filePath2);
```

## Teeing Collector

Teeing Collector is the new collector utility introduced in the Streams API.
This collector has three arguments – `Two collectors` and a `Bi-function`. All input values are passed to each collector and the result is available in the Bi-function. It is a great tool to calculate `mean`.
```java
double mean = Stream.of(1, 2, 3, 4, 5)
    .collect(Collectors.teeing(summingDouble(i -> i), 
    counting(),(sum, n) -> sum / n));
// mean = 3.0
```
## Compact Number Formatting

CompactNumberFormat has been added. It’s designed to represent a number in a shorter form, based on the patterns provided by a given locale.

```java
NumberFormat year = NumberFormat .getCompactNumberInstance(new Locale("en", "US"), 
    NumberFormat.Style.SHORT);
year.setMaximumFractionDigits(2);
System.out.println(year.format(2020));
```
Output:
```java
2.02K
```

## [JEP 325: Switch Expressions (Preview)](https://openjdk.java.net/jeps/325)

I think the most popular feature of Java 12 is Pattern Matching. Let’s compare old and new syntax. In the old code, the syntax looks like the example below:
```java
switch (day) { 
  case MONDAY: 
  case FRIDAY: 
  case SUNDAY: 
    System.out.println(6); 
    break; 
  case TUESDAY: 
    System.out.println(7); 
    break; 
  case THURSDAY:
  case SATURDAY: 
    System.out.println(8); 
    break;
  case WEDNESDAY: 
    System.out.println(9); 
    break;
}
```
With Java 12, the previous code can now be written:
```java
switch (day) { 
  case MONDAY, FRIDAY, SUNDAY -> System.out.println(6); 
  case TUESDAY -> System.out.println(7); 
  case THURSDAY, SATURDAY -> System.out.println(8); 
  case WEDNESDAY -> System.out.println(9);
}
```

It is more compact and doesn’t require break statements.

## [JEP 305: Pattern Matching, for instance of (Preview)](https://openjdk.java.net/jeps/305)

Another preview feature is pattern matching for instanceof. Old code looked like:
```java
Object obj = "Hello World!";
if (obj instanceof String) { 
  String s = (String) obj;
}
```
In the new code, you can check type of variable and assign it to a new variable:
```java
if (obj instanceof String s) { 
  // you can use variable s here
}
```

## [JEP 189: Shenandoah: A Low-Pause-Time Garbage Collector (Experimental)](https://openjdk.java.net/jeps/189)

A new garbage collection (GC) algorithm named `Shenandoah` has been added in Java 12. 
It reduces GC pause times by doing evacuation work concurrently with the running Java threads. 
Pause times with Shenandoah are independent of heap size, meaning you will have the same consistent pause times whether your heap is 200 MB or 200 GB.

## [JEP 230: Microbenchmark Suite](https://openjdk.java.net/jeps/230)

A basic suite of microbenchmarks were added to the JDK source code.


## [JEP 341: Default CDS Archives](https://openjdk.java.net/jeps/341)

Class-Data Sharing (CDS), added in Java 10, has been enabled by default.

# Summary

Java 12 is characterized as a feature release. Personally, I liked bold experiments which were added and syntax improvements.