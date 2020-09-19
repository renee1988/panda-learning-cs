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

A class can provide its consumers with static factory methods or public constructors.

# Advantages of static factory methods

- Unlike constructors, **they have names**.
```java
  // Constructor
  BigInteger(int, int, Random)

  // Static factory method
  BigInteger.probablePrime(int, int, Random)
```
- Sometimes you see class author overloading their constructors whose parameter lists differ only in the order of their parameter types, which is a bad idea → The user of such an API will never remember which constructor is which and end up calling the wrong one.
- Unlike constructors, they are NOT required to create a new object each time they are called.
  - Allows immutable classes to use pre-constructed instances 
  - Allows to cache instances as they are constructed and dispense them repeatedly to avoid creating unnecessary duplicate objects
    - `Boolean.valueOf(boolean)` never creates an object (Flyweight pattern)
    - Greatly improve performance if equivalent objects are requested often, especially if they are expensive to create
  - Allows classes to maintain strict control over what instances exist at any time → instance-controlled classes
    - Instance control allows a class to guarantee that it is a singleton or non-instantiable
    - Instance control allows an immutable value class to make the guarantee that no two equal instances exist (two instances have the same reference)
- Unlike constructors, they can return an object of **any subtype of their return type**.
  - An API can return objects without making their classes public, hiding implementation classes in this fashion leads to a very compact API → interface-based frameworks
  - Interfaces provide natural return types for static factory methods (Example: [Collection framework](https://docs.oracle.com/javase/1.5.0/docs/guide/collections/))
  - Using such a static factory method requires the client to refer to the returned object by **interface** rather than implementation class
- The class of the returned object **can vary** from call to call as a function of the input parameters.
  - The consumer neither knows nor cares about the class of the object they get back from the factory.
- The class of the returned object need not exist when the class containing the method is written → service provider framework
  - Service provider framework is a system decoupling the clients from the implementation of a service:
    - Service interface: represents an implementation
    - Provider registration API: providers use it to register implementations
    - Service access API: clients use it to obtain instances of the service → it is a flexible static factory

# Disadvantages of only providing static factory methods in class

- Main limitation of providing only static factory methods: **Classes without public or protected constructors can’t be subclassed**, which can be a blessing:
  - It encourages programmers to use composition over inheritance
  - It is required for immutable types
- Static factory methods are hard for programmers to find, they don’t stand out in API documentation like constructors do
  - use common naming conventions
    - `from`: type-conversion method: `ClassA.from(instant)`
    - `of`: aggregation method: `ClassA.of(x, y, z)`
    - `valueOf`: `BigInteger.valueOf(123)`
    - `instance` or `getInstance`: returns an instance that is described by its parameters
    - `create` or `newInstance`: returns a new instance
    - `getType`: `Files.getFileStore(path)`
    - `newType`: `Files.newBufferedReader(path)`
