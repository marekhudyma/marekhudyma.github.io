---
layout: post
title: "Java 15 new features"
featured: false
author: marek
categories: Java
type: post
image: '/assets/2020-12-01-java-15-new-features/java15.png'
comments: false
---

<figure>
  <img src="/assets/2020-12-01-java-15-new-features/java15.png" alt="Java 15" />
</figure>

Java 15 was released on March 17, 2020. Let's see its new features. 

# Syntax features

## [JEP 360: Sealed Classes (Preview)](https://openjdk.java.net/jeps/360)

`Sealed classes` feature enables more fine-grained inheritance control. 
It allows to define allowed subtypes. 
In the example below, we define a class `Shape` that permits be extended only by two subtypes: Circle and Rectangle.
The `Circle` class extends `Shape` and is marked as `non-sealed`.
The `Rectangle` class is marked as `sealed`, and permits to be extended only by class `YellowRectangle`.

```java
public sealed class Shape permits Circle, Rectangle {
  // ...
}

public non-sealed class Circle extends Shape {

}

public sealed class Rectangle extends Shape permits YellowRectangle {
  // ...
}
```
`Sealed classes` need to follow the rules:
* Classes that extend `sealed classes` need to be marked as: 
    * `final` - so no other class can extend it, 
    * `sealed` - with list of classes that can extend it. 
    * `non-sealed` - open class for extension to everybody else.
* All permitted subtypes need to belong to the same module as `sealed class`.

Interfaces also can use `sealed` syntax, example: 
```java
public sealed interface Shape permits Circle, Rectangle {
  // ...
}

public non-sealed interface Circle extends Shape {
  // ...
}

public sealed interface Rectangle extends Shape permits YellowRectangle {
  // ...
}
```

One of the places where we can demonstrate the `sealed classes` feature is pattern matching. 
When we use it, the compiler knows that we covered all cases and we do not need to add another part. 
Usually developers were throwing some Exception to highlight that there is some logical problem. 

```java
if (shape instanceof Circle) {
  return ((Circle) shape).getCenter();
} else if (person instanceof Manager) {
  return ((Rectangle) shape).getRectangleCenter();
}
```

In future versions of Java, the client code will be able to use a switch statement instead of if-else [JEP 375: Pattern Matching for instanceof (Second Preview)](https://openjdk.java.net/jeps/375).

## [JEP 384: Records (Second Preview)](https://openjdk.java.net/jeps/384)

`Records` were introduced in `Java 14`, in `Java 15` got a second preview.
It is still preview, but some changes were added:
* Record fields are not modifiable through reflection (finally true immutable object), 
* Native methods are prohibited (`native` is a keyword), 
* Records implement interfaces and work with sealed classes,
* Similar to local classes, records can be local in scope (let's see example below).

`Local records` can be declared and used inside a method:  
```java
void method() {
  record Customer(int id, String name) {};
  Customer customer = new Customer(1, "John");
  // ..
}
```

## [JEP 371: Hidden Classes](https://openjdk.java.net/jeps/371)
Hidden classes are intended for use by frameworks that generate classes at run time and use them indirectly, via reflection. A hidden class may be defined as a member of an access control nest, and may be unloaded independently of other classes.

## [JEP 383: Foreign-Memory Access API (Second Incubator)](https://openjdk.java.net/jeps/383)
Foreign memory access is already an incubating feature of Java 14.
Introduced an API to allow Java programs to safely and efficiently access foreign memory outside of the Java heap.

## [JEP 377: ZGC: A Scalable Low-Latency Garbage Collector (Production)](https://openjdk.java.net/jeps/377) && 
[JEP 379: Shenandoah: A Low-Pause-Time Garbage Collector (Production)](https://openjdk.java.net/jeps/379)

## [JEP 368: Text Blocks (Second Preview)](https://openjdk.java.net/jeps/368)
Text blocks were introduced in Java 13, 14, from Java 15 it became a fully supported product feature.

## [JEP 358: Helpful NullPointerExceptions](https://openjdk.java.net/jeps/358)
Helpful null pointer exceptions introduced in Java 14, became a fully supported product feature.

## Other
The Nashorn JavaScript engine, originally introduced in Java 8, is now removed. 

# Summary 
`Java 15` introduced many production features. It also provided `sealed classes` and iterated over `records`. 
