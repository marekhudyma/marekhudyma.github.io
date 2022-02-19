---
layout: post
title: "Java unsigned integer"
featured: false
author: marek
categories: Java
type: post
image: '/assets/2019-05-01-java-unsigned-integer/binary_notes.jpeg'
comments: false
---

<figure>
  <img src="/assets/2019-05-01-java-unsigned-integer/binary_notes.jpeg" alt="Java unsigned integer header" />
</figure>

# Introduction

In most modern programming languages, there are `signed` and `unsigned` numeric types, eg. `unsigned int` in C++ or `uint` in C#.
Java did not introduce unsigned types! The most probable reason is to make Java simple.
What if we really need to use `unsigned integer` or `unsigned long`? There is a work around that I will explain in this article. Before we do it, let's understand how integer numbers are represented internally.

## Motivation
For many years of development in Java I rarely got a need to use unsigned integers, from technical point of view.
If we loose 1 bit from 32, it is not a big deal. If `Integer` is too small, we can use `Long` with `Long.MAX_VALUE` that is equal to `nine quintillion` (`9223372036854775807`).
When `Long` type is not enough, we can still use `BigInteger` with theoretical unlimited size.

I would divide a `need` of having unsigned integer type to two cases:
* make a logical representation of items, for example to represent number of wheels in the car. 
For most cases unsigned integer is a perfect choice. What would it mean that car has -2 wheels? In fact, we use signed integer for it.
Whereas, I would like to avoid a philosophical discussion `if` we need unsigned integers to properly represent our models.
* technical aspects.

I find it useful to have unsigned integers in 2 areas:
* when we make low level operations, eg. integration with electronic devices, 
* when we need really high performant operations on `unsigned integer` (memory and speed) during mathematical calculations. This was my initial motivation to investigate the topic. 

I believe, that you as a Reader can find more cases where unsigned integer is required.
  
## Binary format

As we know, computer uses [binary format](https://en.wikipedia.org/wiki/Binary_number) to represent numbers.
Each position represents a power of two.

`01` = 0<sup>2</sup> + 1<sup>1</sup> = `1` <br />
`10` = 1<sup>2</sup> + 0<sup>1</sup> = `2` <br />
`11` = 1<sup>2</sup> + 1<sup>1</sup> = `3` <br />

# Binary format in Java

Java uses [two's complement](https://en.wikipedia.org/wiki/Two%27s_complement) to store integers.

## Signed positive numbers
Let's consider how Java stores positive signed numbers. For simplicity, let's say that we have `4` bits numbers.
First bit determines the sign. `0` means that number is positive. To represent the actual value of the number we have `3` another bits.
Examples:

`0011` = `0` on the left means that number is positive <br />
&nbsp;&nbsp;`011` value is equal to: 0<sup>2</sup> + 2<sup>1</sup> + 2<sup>0</sup> = `3` <br />

`0101` = `0` on the left means that number is positive <br />
&nbsp;&nbsp;`101`value is equal to: 2<sup>2</sup> + 0<sup>1</sup> + 2<sup>0</sup> = `5` <br />

## Negative numbers
Negative numbers are much more complicated to represent. Java uses [two's complement](https://en.wikipedia.org/wiki/Two%27s_complement) representation.
The movie below explains how [one's complement](https://en.wikipedia.org/wiki/Ones%27_complement) and [two's complement](https://en.wikipedia.org/wiki/Two%27s_complement) representation works.

<iframe width="560" height="315" src="https://www.youtube.com/embed/4qH4unVtJkE" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

To summarize it:
```
The negative numbers are represented by inverting 1's to 0's and vice versa for all of the bits in a value, then adding 1 to the result.
```

For example: Let's build `-5` from `5` number:

`0101` = `5` <br />
`1010` (inverted values) <br />
`1011` (added 1 to inverted values) = -5 <br />

For 4 bits all the numbers represented in two's complement form looks like:
```
1000 = -8
1001 = -7
1010 = -6
1011 = -5
1100 = -4
1101 = -3
1110 = -2
1111 = -1
0000 =  0
0001 =  1
0010 =  2
0011 =  3
0100 =  4
0101 =  5
0110 =  6
0111 =  7
```

What is worth to mention:
* we have values from `-8` to `+7`, (accordingly: `Integer.MAX_VALUE` = `2_147_483_647`, `Integer.MIN_VALUE` = `-2_147_483_648`) - there is one more negative value than positive.
* in contrast to [Ones' complement](https://en.wikipedia.org/wiki/Ones%27_complement) representation, we have only one `zero value`.

## Adding numbers
One of the biggest advantage of having [two's complement](https://en.wikipedia.org/wiki/Two%27s_complement) form is easy way of adding numbers. It works in the same way as algorithm learned at school, let's follow the example for 4 bits number:
```
   1 1   (carry over)
   0101  (5) 
   1101 (-3)
------------
[1]0010  (2)
```
The result here is `10010`. The carry over made this number 5 bits long. We drop the first bit to make it 4 bits long. The result is `0010` as binary, equal to 2 in decimal form.

# Overflow
Java doesn't do anything with integer overflow for either int or long primitive types and ignores overflow with positive and negative integers.
Example:

If we add `1` to max integer value we will simply overflow to integer min value.
```
 01111111111111111111111111111111 = 2_147_483_647 = Integer.MAX_VALUE
+00000000000000000000000000000001 = 1
---------------------------------
 10000000000000000000000000000000 = -2147483648 = Integer.MIN_VALUE
```

If we subtract `1` from min integer value (in the example `add -1`) we will simply overflow to integer max value.
```
 10000000000000000000000000000000 = -2_147_483_648 = Integer.MIN_VALUE
+11111111111111111111111111111111 = -1 
---------------------------------
 01111111111111111111111111111111 = 2_147_483_647 = Integer.MAX_VALUE
```

JVM will not throw any exception, it will simply execute the operation.

# Unsigned operations
Since java 1.8, there are static helper methods for `Long`, `Integer`, `Short` and `Byte`.

## Conversion unsigned integers

To convert unsigned number from `string` to `int` type, we can use: `Integer.parseUnsignedInt("4000000000", 10);`. 
It will convert number 4 billion (that cannot fit into 31 bits), with radix `10`. 
Internally it will be signed decimal `-294967296`, but unsigned: `4000000000`.
Method `Integer.toUnsignedString(value)` converts binary representation to `string` and treat it as unsigned integer.

Let's analyze the code belowe:
```java
int value = Integer.parseUnsignedInt("4000000000", 10);
System.out.println("decimal value = " + value);
System.out.println("binary representation = " + Integer.toUnsignedString(value, 2));
System.out.println("unsigned decimal representation = " + Integer.toUnsignedString(value));
```
output:
```java
decimal value = -294967296
binary representation = 11101110011010110010100000000000
unsigned decimal representation = 4000000000
```

## Adding unsigned integers

Let's see if we can add two integer numbers and cross `Integer.MAX_VALUE` = `2_147_483_647`.
```java
int value = Integer.parseUnsignedInt("2000000000");
value = value + value;
System.out.println("decimal value = " + value);
System.out.println("binary representation = " + Integer.toUnsignedString(value, 2));
System.out.println("unsigned decimal representation = " + Integer.toUnsignedString(value));
```
output:
```java
decimal value = -294967296
binary representation = 11101110011010110010100000000000
unsigned decimal representation = 4000000000
```
The value `2000000000` (2 billions) almost exeet the `Integer.MAX_VALUE`.
If we double this value, the result will still be in 32 bits, but integer with sign cannot handle it.
It will overflow. That's why we see decimal result `-294967296`.
Whereas in the unsigned format it is `4000000000`.

## Multiplying unsigned integers
Multiplication works in the same way. Instead of adding 2 billions to each other, I multiply it by 2.
```java
int value = 2_000_000_000;;
value = value * 2;
System.out.println("decimal value = " + value);
System.out.println("binary representation = " + Integer.toUnsignedString(value, 2));
System.out.println("unsigned decimal representation = " + Integer.toUnsignedString(value));
```
output:
```java
decimal value = -294967296
binary representation = 11101110011010110010100000000000
unsigned decimal representation = 4000000000
```
In case of multiplying result looks the same as for adding. In decimal format we see overflow (`-294967296`), but in the unsigned decimal representation we see a proper result (`4000000000`).

## Division unsigned integers
Division is also possible with unsigned integers. JDK introduced method: `Integer.divideUnsigned(int dividend, int divisor)`.

```
int value = Integer.parseUnsignedInt("4000000000");
value = Integer.divideUnsigned(value, 2);
System.out.println("decimal value = " + value);
System.out.println("binary representation = " + Integer.toUnsignedString(value, 2));
System.out.println("unsigned decimal representation = " + Integer.toUnsignedString(value));
```
output:
```java
decimal value = 2000000000
binary representation = 1110111001101011001010000000000
unsigned decimal representation = 2000000000
```
In this example we were able to divide 4 billion by 2 in a correct way.

# Alternative solutions
If we do not want to use JDK, [Guava library](https://github.com/google/guava) provides [UnsignedInts](https://guava.dev/releases/31.0-jre/api/docs/com/google/common/primitives/UnsignedInts.html) class.

```java
UnsignedInteger value = UnsignedInteger.valueOf(2_000_000_000);
value = value.times(UnsignedInteger.valueOf(2));
System.out.println(value);
```
output:
```
4000000000
```

# Summary
Officially Java didn't introduce unsigned integer types.
Whereas the workaround was introduced in `Java 8` and determinated developer can make these operations.