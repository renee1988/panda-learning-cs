---
title: 'Item 2: Consider a builder when faced with many constructor parameters'
date: 2020-09-20
permalink: /effective_java/item-1
tags:
  - java
  - full stack
  - best practice
  - back end
  - builder
excerpt: "When to use a builder?"
collection: effective_java
---

Static factories and constructors shared a limitation: **they do not scale well to large numbers of optiona parameters**
- **telescoping constructor pattern** works, but it is hard to write client code when there are many parameters and harder still to read it.
    - Long sequences of identically typed parameters can cause subtle bugs

```java
// Example: telescoping constructor
public class SomeClass {
    private final int a;
    private final int b;
    ...
    private final int f;
    
    public SomeClass(a, b, c) {
        this(a, b, c, 0);
    }
    public SomeClass(a, b, c, d) {
        this(a, b, c, d, 1);
    }
    public SomeClass(a, b, c, d, e) {
        this(a, b, c, d, e, 2);
    }
    public SomeClass(a, b, c, d, e, f) {
        this(a, b, c, d, e, f);
    }
}
```
- **JavaBeans pattern**: you can call a parameterless constructor to create the object and then call setter methods to set each required parameter and each optional parameter of interest.
    - It allows inconsistency, mandates mutability.
    - It precludes the possibility of making a class immutable
    - It requires effort on the part of the programmer to ensure thread safety

## Builder Pattern
Combining the safety of telescoping constructor pattern with the readability of JavaBeans pattern → **Builder**

```java
// Builder pattern
public class SomeClass {
    private final int a;
    private final int b;
    ...
    private final int f;
    
    public static class Builder {
        private final int a;
        private final int b;
        private final int c;

        private int d = 0;
        private int e = 1;
        private int f = 2;
        
        public Builder(int a, int b, int c) {
            this.a = a;
            this.b = b;
            this.c = c;
        }
        public Builder setD(int val) {
            this.d = val;
            return this;
        }
        public Builder setE(int val) {
            this.e = val;
            return this;
        }
        public Builder setF(int val) {
            this.f = val;
            return this;
        }
        
        public SomeClass build() {
            return new SomeClass(this);
        }
    }
    
    private SomeClass(Builder builder) {
        a = builder.a;
        ...
        f = builder.f;
    }
}
```
- The builder pattern simulates named optional parameters
- The builder pattern is well suited to class hierarchies
    - Abstract classes have abstract builders, concrete classes have concrete builders

```java
public abstract class Pizza {
    public enum Topping { HAM, MUSHROOM, ONION, PEPPER, SAUSAGE }
    final Set<Topping> toppings;
    
    abstract static class Builder<T extends Builder<T>> {
        EnumSet<Topping> toppings = EnumSet.noneOf(Toppings.class);
        public T addTopping(Topping topping) {
            toppings.add(Objects.requireNonNull(topping));
            return self();
        }
        abstract Pizza build();
        // Subclasses must override this method to return "this"
        protected abstract T self();
    }
    
    Pizza(Builder<?> builder) {
        toppings = builder.toppings.clone();
    }
}
```
Note: Pizza.Builder is a generic type with a recursive type parameter .
- Along with abstract self method, allows method chaining to work properly in subclasses without casting

```java
public class NyPizza extends Pizza {
    public enum Size { SMALL, MEDIUM, LARGE }
    private final Size size;
    
    public static class Builder extends Pizza.Builder<Builder> {
        private final Size size;
        
        public Builder(Size size) {
            this.size = Objects.requireNonNull(size);
        }
        @Override
        public NyPizza build() {
            return new NyPizza(this);
        }
        @Override
        protected Builder self() {
            return this;
        }
    }
    
    private NyPizza(Builder builder) {
        super(builder);
        size = builder.size;
    }
}
```
A subclass method is declared to return a subtype of the return type declared in the superclass → **covariant return typing**.

## Disadvantages of a builder
In order to create an object, you must first create a builder. While the cost of creating this builder is unlikely to be noticeable in practice. It could be **a problem in performance-critical situations**

## Summary
The builder pattern is a good choice when designing classes whose constructors or static factories would have more than a handful of parameters. Especially if **many of the parameters are optional or of identical type**.