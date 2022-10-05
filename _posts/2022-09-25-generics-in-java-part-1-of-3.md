---
layout: post
title: Generics in Java part 1 of 3
author: borko
description: Generics in Java had been introduced since Java 5. We'll see how they
  contribute to code reusability by adding an extra layer of abstraction over reference
  types. In this part we are going to introduce the motivation for generics, their
  common usage and basic syntax.
categories:
- java
tags:
- java
- generics
image: "/assets/img/posts/2022-09-29-generics-in-java-part-1-of-3/thumbnail.png"
date: 2022-09-25 17:34 +0200
---

![thumbnail](/assets/img/posts/2022-09-29-generics-in-java-part-1-of-3/thumbnail.png)

In this series of articles, we'll discuss Generics in Java.

I will keep updated links to all parts here:

- [Generics in Java part 1 of 3]({% post_url 2022-09-25-generics-in-java-part-1-of-3 %})
- [Generics in Java part 2 of 3]({% post_url 2022-10-05-generics-in-java-part-2-of-3 %})

Generics in Java had been introduced since Java 5. We'll see how they contribute to code reusability by adding an extra layer of abstraction over reference types.

In this part, we are going to introduce the motivation for generics, their common usage, and basic syntax.

---

## A brief overview of Polymorphism

Polymorphism promotes generalization. This brief overview will help us later on in our journey of learning Generics.

In Java (or any other OOP language for that matter) superclasses can be used as polymorphic types for method parameters (including constructors) and can be used as return types of methods.

This allows us to pass subclass objects as arguments and return subclass objects from methods.

```java
class Vehicle {
    private Date dateOfManufacture;
    ...
}

class SportCar extends Vehicle {...}

class Minivan extends Vehicle {...}

class Demo {
    public Vehicle generateVehicle(Boolean isForFamily) {
        if (isForFamily) {
            return new Minivan();
        } 
        else {
            return new SportCar();
        }
    }

    public Date getVehicleDateOfRegistration(Vehicle vehicle) {
        return vehicle.dateOfManufacture;
    }
}
```

Method `generateVehicle` can return any `Vehicle` object or subclass of the `Vehicle` such as `SportCar` or `Minivan`.

And method `getVehicleDateOfRegistration` can be invoked with any `Vehicle` or its subclass object, as we don't care what exact type it is to get its `dateOfManufacture` field.

Interfaces provide even more generalization since they can be implemented by classes from completely different class hierarchies.

```java
interface Printable {
    void print();
}

class Animal implements Printable {
    @Override
    public void print() {
        System.out.println("I am object of Animal class");
    }
}

class Employee implements Printable {
    @Override
    public void print() {
        System.out.println("I am object of Employee class");
    }
}

class Demo {
    public Printable getPrintable(Boolean isHuman) {
        if (isHuman) {
            return new Employee();
        }
        else {
            return new Animal();
        }
    }

    public void printIfNotNull(Printable printable) {
        if (print != null) {
            printable.print();
        }
    }
}
```

As you can see, these classes do not have much in common, but since they share the `Printable` interface, we can use objects of these classes as method arguments or return types where the `Printable` interface is expected.


---

## Motivation for Generics

Programming languages naturally evolve over time and developers themselves are usually the ones who are asking for certain language features. And in most cases, these features are something that is useful for developers but is still missing in a programming language.

The same is true for Generics. To better understand how developers came up with the idea of Generics, we can go through an example.

### Example introduction

Methods in Java natively cannot return multiple values. So you decide to make your own class that will help you achieve this goal. So you create a very simple class `StringIntegerTuple` that just has a constructor and a getter for underlying fields:

```java
public class StringIntegerTuple {
    private final String text;
    private final Integer number;

    public StringIntegerTuple(final String text, final Integer number) {
        this.text = text;
        this.number = number;
    }

    public String getText() {
        return text;
    }

    public Integer getNumber() {
        return number;
    }
}
```

Now you can use this class as a return type of method. This allows you to return `String` and `Integer` to the calling method.

```java
public StringIntegerTuple calculateAndReturnResultWithComment() {
    Integer result = 10;
    String comment = "Result is ten";
    return new StringIntegerTuple(comment, result);
}
```

> In general, you could use subtypes instead of exact types `String` and `Integer`, but these classes are `final`, which means they cannot have subtypes. Still, for simplicity we chose `String` and `Integer` in this example.
{: .prompt-info}

You like this approach and you start building similar classes:

- StringDoubleTuple
- StringFloatTuple
- IntegerIntegerTuple
- ...

Now you notice that the code base starts to grow purely because our helper classes use hard-coded types, so you would like to generalize it somehow.

### Generalize Tuple without generics

At this moment it's obvious that you would like a way to generalize these classes.

So you come up with a following idea: let's use the superclass of all the classes, in other words, use `Object` class instead of both types for the tuple:

```java
public class Tuple {
    private final Object firstMember;
    private final Object secondMember;

    public Tuple(final Object firstMember, final Object secondMember) {
        this.firstMember = firstMember;
        this.secondMember = secondMember;
    }

    public Object getFirstMember() {
        return firstMember;
    }

    public Object getSecondMember() {
        return secondMember;
    }
}
```

And it would be used like this:

```java
public Tuple calculateAndReturnResultWithComment() {
    Integer result = 10;
    String comment = "Result is 10";
    return new Tuple(comment, result);
}
```

Now you are happy that all those classes are gone. But wait for a second, we just introduced some problems here.

### The downside of generalization with Object class

As we are dealing with an `Object` class, to make use of it we need to do explicit casting. And that produces error-prone code since there is no type of safety anymore!

Let's say that some class method is returning `Tuple` with types `String` and `Date` (java.util.Date):

```java
public static Tuple generateStringDateTuple() {
    String stringValue = "";
    java.util.Date date = new java.util.Date();
    return new Tuple(stringValue, date);
}
```

Now consumers need to do the explicit cast of the return types

```java
public static void firstConsumer() {
    Tuple tuple = generateStringDateTuple();
    String stringValue = (String) tuple.firstMember;
    java.util.Date date = (java.util.Date) tuple.secondMember;
}

public static void secondConsumer() {
    Tuple tuple = generateStringDateTuple();
    String stringValue = (String) tuple.firstMember;
    // next line will throw the ClassCastException at runtime
    java.sql.Date date = (java.sql.Date) tuple.secondMember; 
}
```

The second consumer will throw `ClassCastException` because it uses the wrong type and therefore secondMember cannot be successfully type casted.

So, to summarize the downsides of this approach:

- loses type safety
- requires explicit casting
- introduces runtime errors

---

## Generics example

> Generics is purely a compile-time concept. There are no generics in runtime! We'll discuss this later on when we are talking about type erasure.
{: .prompt-danger}

Now that we've seen the pre-generics generalization approach, we can switch to a Generics feature of Java to solve this design problem.

For now, don't worry about how exactly Generics works, we will cover it later on. But first, we will show an example and what benefits we get from this approach.

With Generics in place, we can rewrite the class as follows:

```java
public class Tuple<T, U> {

    private final T firstMember;
    private final U secondMember;

    public Tuple() {
        firstMember = null;
        secondMember = null;
    }

    public Tuple(final T firstMember, final U secondMember) {
        this.firstMember = firstMember;
        this.secondMember = secondMember;
    }

    public T getFirstMember() {
        return firstMember;
    }

    public U getSecondMember() {
        return secondMember;
    }
}
```

The usage is now different. Our producer would look like this:

```java
public static Tuple<String, java.util.Date> generateStringDateTuple() {
    String stringValue = "";
    java.util.Date date = new java.util.Date();
    return new Tuple<String, java.util.Date>(stringValue, date); 
}
```

Let's take a look at the same example we did for a Tuple without Generics:

```java
public static void firstConsumer() {
    Tuple<String, java.util.Date> tuple = generateStringDateTuple();
    String stringValue = tuple.firstMember;
    java.util.Date date = tuple.secondMember;
}

public static void secondConsumer() {
    Tuple<String, java.util.Date> tuple = generateStringDateTuple();
    String stringValue = tuple.firstMember;
    // next line will throw a compilation error
    java.sql.Date date = tuple.secondMember;
}
```

This code will not compile, since there is type check at compile time and most probably your editor will give you a warning for 3rd line in the `secondConsumer` method even before you try compilation.

Note how there is no more explicit casting for the type of object when using Generics.

It's very easy now to create Generics `Tuple` with different types for their fields:

```java
Tuple<String, Double> stringDoubleTuple = new Tuple<String, Double>();
Tuple<String, Float> stringFloatTuple = new Tuple<String, Float>();
Tuple<Integer, Integer> integerIntegerTuple = new Tuple<Integer, Integer>();
```

Now, to summarize the upsides of this approach:

- type safety at compile time
- explicit casting is no longer required
- cleaner code
- code is generic (you can instantiate the same class with different type parameters)

### Java Collections Framework

Some of the most used generics are the ones from Java Collections Framework.

Prior to Java 5, Java Collections Framework used the `Object` as a polymorphic type. Even keys and values in maps were Object types. But since Java 5, Collections Framework switched to Generics.

You are probably already using them, as they are the foundation of Data Structures in Java. Here is the list of the most used ones:

- `Collection<E>`
- `List<E>`
- `Map<K, V>`
- `Set<E>`
- `Queue<E>`

---

## Generics basic syntax and terminology

Let's take a closer look at the basic syntax and terminology for the generics.

### Generic type

> A **Generic type** is a **class** or an **interface** with **type parameters** defined in the declaration
{: .prompt-info}

```java
class ClassName <T1, T2, T3, ...> {...}
```

Syntax of the generic type is simple, it's the same as an ordinary class or an interface with something called `type parameters`. These are listed within `<` and `>`. In our example that would be **T1**, **T2**, **T3**... You can use any identifier, it does not have to start with **T**, we could use something like `<X,Y,Z>`.

The naming convention for type parameters is to use single uppercase letters (although it's not mandatory for your code to work).

In many cases type parameter points to its nature, for example:
- `E` for element
- `K` for key
- `V` for value (check Java Collections Framework)

Now, these type parameters can be used in a class (or interface) to represent the type of:

- instance variables
- parameters
- local variables
- return types

In our example of a Generic class, we can see that `T` and `U` are used as:

- type of the class fields for firstMember and secondMember
- type of the parameters for the constructor
- the return type of the getters

### Type Parameter

Now that we've seen how generic type is constructed, let's look at how `type parameters` can be used when constructing instances of the generic classes.

Back to our example:

```java
Tuple<String, java.util.Date> stringDateTuple = new Tuple<String, java.util.Date>();
Tuple<String, String> stringStringTuple = new Tuple<String, String>();
Tuple<Integer, String> integerStringTuple = new Tuple<Integer, String>();
Tuple<String, List<String>> stringListTuple = new Tuple<String, List<String>>();
```

In fact, we can make this even simpler, because Java can infer types of parameters on the right-hand side. This `<>` notation is called `Diamond notation` and can be used from Java 7 onwards.

```java
Tuple<String, java.util.Date> stringDateTuple = new Tuple<>();
Tuple<String, String> stringStringTuple = new Tuple<>();
Tuple<Integer, String> integerStringTuple = new Tuple<>();
Tuple<String, List<String>> stringListTuple = new Tuple<>();
```

Did you notice something interesting here? The type parameter itself can be another generic type 🙂. In this case it's `List<String>`

There are two important distinctions in the terminology of Generics:

- **Generic type** is a type with `type parameters`, as we've already seen
- **Parameterized type** on the other hand is the one that we get once we change type parameters with actual types, in other words `type arguments`

### Generics basic syntax summary

It's much clearer when you check out the example:

- `Tuple<T, U>` - generic type
- `Tuple<Integer, String>` - parameterized type


- `T` and `U` - type parameters
- `Integer` and `String` - type arguments

---

## Subtyping Generic type

We can subtype Generic type in a very similar way as we would do if we had regular classes and interfaces.

Let's start with an interface:

```java
interface Pair<F, S> {
    F getFirstMember();
    S getSecondMember();
}
```

As you can see, it's defining getters for first and second members and is generic. In other words, it doesn't require any specific type, but instead, it defines methods that should exist and their return types are generic ones.

Next, we can take the same `Tuple` class we are working with and change its definition:

```java
public class Tuple<T, U> implements Pair<T, U> {
    ...
}
```

Type parameters are intentionally different in definitions of interface and class, so we can show how they have a meaning only in the context they are used. You can think of them as placeholders for real types.

What I want to say is that these **type parameters** `F` and `S` for interface `Pair` has a meaning in that interface definition only. In order to transfer type parameters from the Generic class that implements this interface, it's important to use the same type parameters (in this case `T` and `U`).

Subtyping the Generics in last example allows us to reference an instance of `Tuple` as an interface `Pair`:

```java
Pair<String, java.util.Date> stringDateTuple = new Tuple<>();
```

And here is how we got ourselves polymorphism with generics 🤯

> Another term for Generics is Parametric Polymorphism
{: .prompt-info}
