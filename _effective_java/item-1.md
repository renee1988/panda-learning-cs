---
title: 'Item 1: Static Factory Methods'
# date: 2020-09-20
# permalink: /effective_java/item-1
# tags:
#   - Java
#   - FullStack
#   - BestPractice
#   - BackEnd
excerpt: "Static Factory methods vs. public constructors"
collection: effective_java
---

# Static Factory methods vs. public constructors

A class can provide its consumers with static factory methods or public constructors.

## Advantages of static factory methods

- Unlike constructors, **they have names**.
```java
  // Constructor
  BigInteger(int, int, Random)

  // Static factory method
  BigInteger.probablePrime(int, int, Random)
```
