---
title: 'Item 13: Override clone judiciously'
date: 2020-09-20
permalink: /effective_java/item-13
tags:
  - java
  - full stack
  - best practice
  - back end
  - dependency injection
excerpt: "What is Cloneable? Why do we prefer copy constructor or factory?"
collection: effective_java
---

## `Cloneable` interface
The `Cloneable` interface was intended as a mixin interface for classes to advertise that they permit cloning.
- It fails to serve this purpose
- Its primary flaw is that it lacks a `clone` method and `Object`’s `clone` method is `protected`.
- You can’t call clone on an object without resorting to reflection, even reflective invocation may fail since it doesn’t guarantee that the object has an accessible clone method.

### What does `Cloneable` do?
It determines the behavior of Object’s protected clone implementation:
- If a class implements `Cloneable`, `Object`’s clone method returns a field-by-field copy of the object, otherwise it throws `CloneNotSupportedException`.
- In practice, a class implementing `Cloneable` is expected to provide a properly functioning public `clone` method.

The general contract for the clone method is weak:
- `x.clone() != x`
- `x.clone().getClass() == x.getClass()`
- `x.clone().equals(x)`

If a class’s clone method returns an instance that is **not** obtained by calling `super.clone` but by calling a constructor, the compiler won’t complain, but if a subclass of that class calls `super.clone`, the resulting object will have the wrong class, preventing the subclass from clone method from working properly.

**Immutable classes should never provide a clone method.**

**If an object contains fields that refer to mutable objects, you must ensure that it does no harm to the original object and that it properly establishes invariants on the `clone`.**
```java
// Example: Stack
// In order for the clone method on Stack to work
// properly, it must copy the internals of the stack.
public class Stack {
    private Object[] elements;
    ...
    @Override
    public Stack clone() {
        try {
            Stack result = (Stack) super.clone();
            // Call clone recursively on the elements array
            result.elements = elements.clone();
            return result;
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }
}
```
Note: the above implementation of `clone` would not work if the elements field were `final`. → **The `Cloneable` architecture is incompatible with normal use of final fields referring to mutable objects.**

## CloneNotSupportedException
- Public clone methods should omit the `throws` clause, as methods that don’t throw checked exceptions are easier to use.
- **For class inheritance, the class should not implement Cloneable**
    - You may provide a properly functioning protected clone method that throws CloneNotSupportedException
```java
// clone method for extendable class not supporting Cloneable
@Override
protected final Object clone() throws CloneNotSupportedException {
    throw new CloneNotSupportedException();
}
```

## Summary
- All classes that implement `Cloneable` should override clone with a public method whose return type is the class itself.
    - this method should first call super.clone, then fix any fields that need fixing (copying any mutable objects that comprise the internal deep structure of the object and replacing the clone’s references to these objects with references to their copies)
    - if the class contains only primitive fields or references to immutable objects, then it is likely the case that no fields need to be fixed
- A better approach to object copying is to provide a **copy constructor** or **copy factory**.
```java
// Copy constructor
public Yum(Yum yum) {...}

// Copy factory
public static Yum newInstance(Yum yum) {...}
```
- A copy constructor or copy factory can take an argument whose type is an interface implemented by classes. → Interface-based copy constructors and factories known as **conversion constructors** and **conversion factories**.
- Allow client to choose the implementation type of the copy rather than forcing the client to accept the implementation type of the original.
- For example, suppose you have a `HashSet hashSet`, and you want to copy it as a `TreeSet`. With conversion constructor, you can have `new TreeSet<>(hashSet)`
- Exception: arrays are best copied with the `clone` method.