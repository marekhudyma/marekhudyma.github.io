---
layout: post
title: "Java String.hashCode() equal to zero"
categories: JDK,Java,String
---

In this article, you will find information on:
* interesting behavior of Java String.hashCode() method, when returned value is equal to zero.

Code with performance benchmarks you can find here [here]:(https://github.com/marekhudyma/string.hashCode).

# Introduction
Every object in Java return hashCode as an `integer` value. It is used in many data structures like: HashMap, HashTable, HashSet etc.
How is String.hashCode() implemented? It can be represented as the simplified pseudo-code below:
```
public int hashCode() {
    if (hash == 0) {
        hash = s[0]*31^(n-1) + s[1]*31^(n-2) + … + s[n-1]
    }
    return hash;
}
```
Where:
* s[i] – is the ith character of the string
* n – is the length of the string
* ^ – indicates exponentiation

String is heavily used in JDK. It is immutable, so it doesn't make sense to compute it every time, when the method is invoked. 
String stores computed hashCode value in the variable. If the cached value (`h`) is equal to zero, the computation is executed. 
 
What will happen if the computed hashCode is equal to zero ? Finally, it is a normal integer value. According to the algorithm, it will compute a hashCode again. In this case, operations on a hashMap should be much slower. 

# Benchmark 
Let's run JMH benchmark to check performance. I ran benchmark below: 
```
    @Benchmark
    @BenchmarkMode(Mode.All)
    @OutputTimeUnit(TimeUnit.MICROSECONDS)
    public void performanceTestOfHashMap(Blackhole blackhole) {
        blackhole.consume(underTest.put(myString, 1));
    }
```
with two texts: 
1. `"normalString"` hashCode is equal `151224280`
2. `"Airlia unhallow"` hashCode is equal `0`.
        
JMH benchmark has shown that insertion the time to hashMap was equal! It was shown that the calculation of hashCode of short Strings is incredibly fast. 

I decided to find a long String with hashCode equal to zero. I took the standard `Lorem ipsum` text and added an integer at the end, as long as I found a String with hashCode = 0. 

Warning: there is number `72098087` at the end. 
```
"Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum. Curabitur pretium tincidunt lacus. Nulla gravida orci a odio. Nullam varius, turpis et commodo pharetra, est eros bibendum elit, nec luctus magna felis sollicitudin mauris. Integer in mauris eu nibh euismod gravida. Duis ac tellus et risus vulputate vehicula. Donec lobortis risus a elit. Etiam tempor. Ut ullamcorper, ligula eu tempor congue, eros est euismod turpis, id tincidunt sapien risus a quam. Maecenas fermentum consequat mi. Donec fermentum. Pellentesque malesuada nulla a mi. Duis sapien sem, aliquet nec, commodo eget, consequat quis, neque. Aliquam faucibus, elit ut dictum aliquet, felis nisl adipiscing sapien, sed malesuada diam lacus eget erat. Cras mollis scelerisque nunc. Nullam arcu. Aliquam consequat. Curabitur augue lorem, dapibus quis, laoreet et, pretium ac, nisi. Aenean magna nisl, mollis quis, molestie eu, feugiat in, orci. In hac habitasse platea dictumst.72098087"
```

I run benchmark again: 
1. with standard `Lorem ipsum` text. 
2. with standard `Lorem ipsum` + `72098087`, so hashCode was equal to zero.
This time, the `unlucky String` was `110 times` slower! This is a significant difference!
  
# Frequency of String.hashCode() = 0
I wanted to check, how frequently can we find a String with hashCode equal zero? I took a list of [english words]( https://github.com/dwyl/english-words) (more than `450_000`) and none of the work had hashCode equal zero!
I was surprised; I thought that it would appear more frequently. 
I decided to write a program to find all bigrams (combinations of 2 words). I found 48:
```
Airlia unhallow
Alphonsa Flavobacterium
Anschluss traction
antiministerially precommuning
Balaenicipites kirbies
belly-cheer thinghood
bequirtle zorillo
BRC Kniphofia
Cchaddie gospodipoda
chronogrammic schtoff
Chronotron Platonically
chut true-false
contusive cloisterlike
creashaks organzine
Dema subadvocate
desmography prememoranda
docoglossan pre-entertain
drumwood boulderhead
electroanalytic exercisable
EMT devilward
favosely nonconstruable
footstone vizored
forepost cut-work
Grignolino forehead's
Karry inhabitate
laccaic dephase
misnames Roquefort
Nollie glisky
nonfeudal premodel
oncet incurtain
overstrain CCIR
Periapis watchfree
plumoseness unimpoisoned
polariscopy urochloralic
pollinating sandboxes
Poplarism Satanophil
purchase superimpersonal
rencountering pachyderm
revolvingly admissable
sphenopetrosal physicianless
suicidical pennatisect
tapper well-winded
tobogganist entreasuring
toxity fizzes
unreprovableness isolysin
untended nonfiduciaries
wheep purificatory
world-diminishing five-page
xanthopsydracia so-named
```

# String improvement
How could the String.hashCode() be improved? One of the solutions is adding a boolean flag. Base on that, the String would determine if calculations should be executed. 
The price for it is storing the boolean flag in every String object. 
Overall speed that we gain is not so big. From a statistical point of view, `unlucky Strings` happen once per 4_294_967_295 Strings. Worst case scenario, JVM will deal with an unlucky String being a little bit slower. 
I think JVM developers were fully aware of it, that's the reason of their implementation. 

# Security
I think this fact can be interesting from a security point of view. If you know that a system is written in Java (JVM based), you can suspect that HashMap / HashSet is used in the implementation. 
You can send very long `unlucky Strings` to the system and cause a performance degradation. What's more funny, it would be extremely difficult to find such a vector of attack. 
[Dos attack](https://en.wikipedia.org/wiki/Denial-of-service_attack) that uses `unlucky Strings` can easily kill our CPU making it calculate hashCode of String all the time. 
The biggest effect would be in systems, which use a lot of HashSet/HashMaps or HashSet/HashMap with a big number of Strings inside.

# Summary
As JVM developers, we need to be aware of String.hashCode() implementation. In most cases, it is just curiosity. I wish it would never cause production issues. 