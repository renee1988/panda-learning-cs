---
title: 'Item 5: Prefer dependency injection to hardwiring resources'
date: 2020-09-20
permalink: /effective_java/item-5
tags:
  - java
  - full stack
  - best practice
  - back end
  - dependency injection
excerpt: "What the heck is dependency injection?"
collection: effective_java
---

Many classes depend on one or more underlying resources.

Example: Spell checker on a dictionary

## Bad implementation: static utility classes
```java
// Inappropriate use of static utility - inflexible and untestable
public class SpellChecker {
    private static final Lexicon dictionary = ...;
    
    private SpellChecker() {}
    
    public static List<String> suggestions(String typo) {...}
}
```

## Bad implementation: singletons
```java
// Inappropriate use of singleton - inflexible and untestable
public class SpellChecker {
    private static final Lexicon dictionary = ...;
    
    private SpellChecker(...) {}
    
    public static SpellChecker INSTANCE = new SpellChecker(...);
    
    public static List<String> suggestions(String typo) {...}
}
```

## Why are singletons and static utility classes bad?
Both of singletons and static utility classes assume there is only one dictionary worth using. In general, **singletons and static utility classes are inappropriate for classes whose behavior is parameterized by an underlying resource**.

## Dependency injection
To support multiple instances of the class, you can pass the resource into the constructor when creating a new instance → **dependency injection**.
- Example: SpellChecker, dictionary is a dependency injected into the spell checker when it is created.
```java
// Dependency injection provides flexibility and testability
// You can mock the dictionary however you desire in your tests
public class SpellChecker {
    private final Lexicon dictionary;
    
    public SpellChecker(Lexicon dictionary) {
        this.dictionary = Objects.requireNonNull(dictionary);
    }
    
    public List<String> suggestions(String typo) {...}
}
```
- Dependency injection works with an arbitrary number of resources and arbitrary dependency graphs.
- It preserves immutability, so multiple clients can share dependent objects.

## Dependency injection framework
For large projects, they contain thousands of dependencies, to eliminate the clutter → use a **dependency injection framework**.