---
layout: post
title: Typing systems explained
date: 2024-07-02 20:45 +0200
author: borko
description: "Beginner friendly introduction to Duck Typing, Structural Typing, and Nominal Typing."
categories: [javascript, typescript, java, go]
tags:
- javascript
- typescript
- java
- go
image: "/assets/img/posts/2024-07-02-typing-systems-explained/thumbnail.png"
---

In the realm of programming languages and type systems, understanding how types are enforced and interpreted is crucial for writing robust and maintainable code. Three common approaches to type checking are Duck Typing, Structural Typing, and Nominal Typing. Each has its own philosophy and use cases, influencing how programmers design and interact with code. This article explores these three typing systems, comparing their characteristics, advantages, and disadvantages.


![Typing systems explained](/assets/img/posts/2024-07-02-typing-systems-explained/thumbnail.png){: .shadow style="width: 90%" }

## Nominal Typing

### Definition

**Nominal Typing** is based on explicit declarations and names. Two types are compatible if they have been declared to be so, usually by sharing a common ancestor or through explicit type declarations.

### Characteristics

- **Name-based**: Type compatibility is determined by explicit type names and declarations.
- **Static**: Typically used in statically typed languages.
- **Explicit relationships**: Types must be explicitly declared to be related.

### Advantages

- **Safety**: Ensures type safety through explicit declarations.
- **Readability**: Improves code readability and maintainability with clear type definitions.
- **Tooling**: Better support from IDEs and development tools for type checking and code completion.

### Disadvantages

- **Verbosity**: Requires more boilerplate code with explicit type declarations.
- **Rigidity**: Less flexible in terms of reusing code and adapting to changes.

### Languages

Java and C# use nominal typing.

```java
interface Quacks {
    void quack();
}

class Duck implements Quacks {
    String name;

    public void quack() {
        System.out.println("Quack");
    }
}

class Person implements Quacks {
    String name;

    public void quack() {
        System.out.println("I'm quacking like a duck");
    }
}

public class Main {
    public static void main(String[] args) {
        Quacks d = new Duck();
        Quacks p = new Person();
        
        makeItQuack(d);  // Outputs: Quack
        makeItQuack(p);  // Outputs: I'm quacking like a duck
    }

    // it can accept Person or Duck because they both implement Quacks interface
    public static void makeItQuack(Quacks duck) {
        duck.quack();
    }

    // on the other hand, this method cannot accept Person object, despite the
    // fact that shapes of the Person and Duck classes are the same
    public static String getDucksName(Duck duck) {
        return duck.name;
    }
}
```


## Structural Typing

### Definition

**Structural Typing** determines type compatibility and equivalence based on the structure (or shape) of the types, rather than their explicit declarations. If two types have the same shape, they are considered compatible.

### Characteristics

- **Shape-based**: Types are compatible if they have the required structure.
- **Static or dynamic**: Can be used in both statically and dynamically typed languages.
- **Implicit compatibility**: No need for explicit declarations of type relationships.

### Advantages

- **Interoperability**: Facilitates easy interaction between different types as long as they share the same structure.
- **Flexibility**: Supports code reuse and generic programming.

### Disadvantages

- **Complexity**: Type errors can sometimes be harder to trace due to implicit type relationships.
- **Maintenance**: Changes in type structure can have wide-reaching effects.

### Languages

Typescript and Go use structural typing.

```typescript
class Duck {
    quack() {
        console.log("Quack");
    }
}

class Person {
    talk() {
      console.log("I speak English");
    }

    quack() {
        console.log("I'm quacking like a duck");
    }
}

function makeItQuack(duck: Duck) {
    duck.quack();
}

function makeItTalk(person: Person) {
    person.talk();
}

let d = new Duck();
let p = new Person();

makeItQuack(d);  // Outputs: Quack
makeItQuack(p);  // Outputs: I'm quacking like a duck

makeItTalk(p); // Outputs: I speak English
makeItTalk(d); // Compilation error
```

## Duck Typing

### Definition

**Duck Typing** is a concept in dynamic typing where an object's suitability is determined by the presence of certain methods and properties, rather than the object's actual type. 

> The name is derived from the saying:
> 
> "*If it looks like a duck and quacks like a duck, it probably is a duck.*"
{: .prompt-info }


### Characteristics

- **Dynamic**: Types are checked at runtime.
- **Behavior-focused**: If an object can perform the required actions, it is considered of the correct type.
- **Flexible**: Allows for more flexible and less verbose code.


### Advantages

- **Ease of use**: Reduces the need for explicit type declarations.
- **Flexibility**: Supports rapid prototyping and dynamic code changes.
- **Less boilerplate**: Minimizes the need for type annotations.

### Disadvantages

- **Runtime errors**: Type errors are caught at runtime, which can lead to unexpected failures.
- **Lack of IDE support**: Less assistance from development tools in terms of code completion and type checking.
- **Reduced readability**: Code can become harder to understand without explicit type information.

### Languages

Python, Javascript and Ruby are prominent languages that use duck typing.

```javascript
class Duck {
    quack() {
        console.log("Quack");
    }
}

class Person {
    talk() {
      console.log("I speak English");
    }
    
    quack() {
        console.log("I'm quacking like a duck");
    }
}

function makeItQuack(duck) {
    // Here, we are not checking for the type of the object, 
    // only if it has a method named 'quack'
    duck.quack();
}

function makeItTalk(person) {
    // Same here, which in our case will cause runtime errors
    person.talk();
}

let d = new Duck();
let p = new Person();

makeItQuack(d);  // Outputs: Quack
makeItQuack(p);  // Outputs: I'm quacking like a duck

makeItTalk(p); // Outputs: I speak English
makeItTalk(d); // Runtime error: person.talk is not a function
```


## Conclusion

Understanding the differences between Duck Typing, Structural Typing, and Nominal Typing is essential for choosing the right approach for a given project. Duck Typing offers flexibility and simplicity, making it ideal for dynamic environments and rapid prototyping. Structural Typing provides a balance of flexibility and safety, suitable for projects that require a middle ground. Nominal Typing ensures strong type safety and readability, ideal for large, complex systems that benefit from explicit type declarations and relationships.

Each typing system has its own strengths and weaknesses, and the choice often depends on the specific requirements of the project and the preferences of the development team. By understanding these differences, developers can make informed decisions to leverage the right type system for their needs. 

