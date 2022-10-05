---
layout: post
title: Generics in Java part 2 of 3
author: borko
description: Generics in Java had been introduced since Java 5. We'll see how they
  contribute to code reusability by adding an extra layer of abstraction over reference
  types. In this part we are going to introduce the concept of type erasure, invariance and restrictions of generics
categories:
- java
tags:
- java
- generics
image: "/assets/img/posts/2022-10-05-generics-in-java-part-2-of-3/thumbnail.png"
date: 2022-10-05 23:27 +0200
---

![thumbnail](/assets/img/posts/2022-10-05-generics-in-java-part-2-of-3/thumbnail.png)

This article is the second part of the series `Generics in Java`.

I will keep updated links to all parts here:

- [Generics in Java part 1 of 3]({% post_url 2022-09-25-generics-in-java-part-1-of-3 %})
- [Generics in Java part 2 of 3]({% post_url 2022-10-05-generics-in-java-part-2-of-3 %})

Now that you know some basic syntax for Generics, as well as the motivation behind it, we can continue our journey to further expand our knowledge of Generics.

In this part, we are going to introduce the concept of type erasure, invariance, and restrictions of generics.

---

## Raw Generic Types

We already mentioned that Generics is a purely compile-time feature of the Java programming language, meaning the Generic types do not exist in the runtime.

To keep Java backward compatible with the old versions, Generics allow you the usage of so-called Raw types.

Imagine that you were going to use a library that is written in Java 8, but your code is written in Java 4. We know that Generics appeared in Java 5, so you cannot use Generic types directly.

In this case, you can use Generic types without **type arguments** and still be able to include that library.

> **Raw type** is a Generic type without type arguments.
{: .prompt-info}

Let's show this with an example.

We know that the Collections framework has Generic types. One such example is the `List` interface, which takes `E` as a `type parameter`.

You already know by now how to create `List` of `Integer` element type:

```java
List<Integer> list = new ArrayList<>();
```

But you are not required to provide a `type argument`, you can just leave it empty like this:

```java
List list = new ArrayList();
```

What happens here is that the `Object` type will be used as a `type argument`.

> If you have multiple type parameters for your Generic class, you cannot omit just one `type argument`. You must either provide all `type arguments`, or omit them all (use Raw Generic type).
{: .prompt-warning }

This brings us back to our first solution in making some class generic - by using the `Object` type in a class. Compile time type safety is lost by using Raw Generics. Needless to say, this approach should be avoided.

> **Using raw types is dangerous!**
{: .prompt-danger}


### Exceptions to not using Raw types

We already said that Raw types should be avoided, but sometimes there are cases in which you don't have a choice but to use them:

- class literals (there is no `List<String>.class` per se, because generic types do not exist in runtime, so we must use `List.class` instead)
- `instanceof` operator (same reason as for class literals)

---

## Type Erasure

Finally, we are going to look at how generics really work under the hood. What does it mean that generics are only compile-time features and they do not exist in runtime?

During compilation, the Java compiler validates types in our code to make sure we don't use Generic types in the wrong way. This is the point where we would get a compilation error if we misuse types.

After the `type check` step, all type arguments are erased. This leaves us with an `Object` as a type where we had type parameters, just as we would have in a case we used a `Raw type`.

In our `Tuple` example, after the compilation the class would essentially behave like this:

```java
public class Tuple {

    private final Object firstMember;
    private final Object secondMember;

    public Tuple() {
        firstMember = null;
        secondMember = null;
    }

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

Now you may ask but if types are replaced by `Object`, what happens to all places in the code where we used this Generic type? Are we going to lose the information about the type argument that is used when this class was instantiated?

Well, it turns out that the compiler will insert an explicit type cast in all places that were using our generics type, so it will behave like this:

```java
public static void firstConsumer() {
    Tuple tuple = generateStringDateTuple();
    String stringValue = (String) tuple.firstMember;
    java.util.Date date = (java.util.Date) tuple.secondMember;
}
```

To summarize, the compiler feature `Type Erasure` is doing two things after compilation:
- erases all type parameters of generics, leaving `Object` as the type where type arguments were used
- inserts explicit casting in places where generic types were used

> In the final part of this series we will show that we can force Java compiler to use some type other than `Object` when doing type erasure by something called `bounds`.
{: .prompt-info}

---

## Invariance in Java Generics

This section contains a bit of theory on the type systems, so feel free to skip it if you want. Here are the key takeaways from this section:

- Java Generics are `invariant`, meaning you cannot substitute supertype or subtype of the type argument.
- Java Arrays are `covariant`, thus allowing for some type casting errors to slip to runtime
- Because of these facts, it's recommended to use Generics instead of Arrays in Java when dealing with collections

If you want to learn more details about `invariance` and `covariance`, just keep on reading 🙂.

### A brief overview of Variance

Let's now discuss an important topic of `variance` in Java. This will get more theoretical, but you will find it quite rewarding as it deepens your understanding of the nature of the `Generics` in Java.

As you already know, a type system in Java supports subtyping. It allows us to substitute a type with any of its subtypes.

But what happens if we have some type transformations, for example, `arrays` and `generics`? Do transformed types keep the same relations as their underlying types? Or do they somehow change that relation?

That behavior is defined by the `variance`. Each `type transformation` can have different `variance` and we'll show an example soon.

> It's important to note that not all programming languages have the same `variance` for the same `type transformations`, but the concept is the same accross all languages. To give an example, in the **OCaml** programming language the `list` type constructor is `covariant`, whereas in **Java** the `List` generic type constructor is `invariant`.
{: .prompt-danger}

### Types of variance

Let's say that we have a type `A` and it is a subtype of type `B`. Also, we'll denote subtype relation with a `≤` symbol.

If `f` is a `type transformation`, we can say the following:

- `f` is `covariant` if `A ≤ B` implies that `f(A) ≤ f(B)`
- `f` is `contravariant` if `A ≤ B` implies that `f(B) ≤ f(A)`
- `f` is `bivariant` if both of these apply (i.e., if `A ≤ B`, implies that `f(A) ≡ f(B)`)
- `f` is `variant` if `f` is `covariant`, `contravariant` or `bivariant`
- `f` is `invariant` or `nonvariant` if not `variant`

We'll cover just what is relevant to our understanding of the **Generics** since this topic is huge and would require a series of articles for itself.


### Java Array vs Java Generic

In Java, we have the following:

- `Generics` are `invariant`
- `Arrays` are `covariant`

#### Generics are invariant

Let's walk through an example to understand the implications of these statements.

We'll take `Animal` and `Dog` types.

`Dog≤Animal` - we read this: `Dog` is subtype of `Animal`.

Since `Generics` are `invariant`, we know that:

- `List<Dog>` is not a subtype of a `List<Animal>`
- `List<Animal>` is not a subtype of a `List<Dog>`

Let's see what would happen if the Generics were not invariant:

```java
void go(ArrayList<Animal> animals) {
    animals.add(new Cat());
}

go (new ArrayList<Dog>());
```

The `ClassCast Exception` would be thrown in runtime, because we cannot cast `Cat` to a `Dog`, as we could to `Animal`.

However, for the same `type argument`, inheritance of generic types is allowed:

`ArrayList<Dog> ≤ List<Dog>` - `ArrayList<Dog>` is a subtype of `List<Dog>`, meaning that `ArrayList<Dog>` can be substituted where `List<Dog>` is expected.

If you think about it, it makes perfect sense. Generic types are transformed to Raw types after compilation and they should behave in the same way as non-Generic types, meaning they should be `covariant` as simple types are `covariant`.

Since type parameters for Generics are invariant in Java, we can say that Generics have much stricter type safety than Arrays, which we'll see in a moment.

#### Arrays are covariant

On the other hand, since `Arrays` are `covariant`:

`Dog[] ≤ Animal[]` - an array of the `Dog` objects is a subtype of an array of the `Animal` objects

Because of that, we can do:

```java
Animal[] animals = new Dog[1];
```

The same example as earlier:

```java
void go(Animal[] animals) {
    animals.add(new Cat());
}

go (new Dog[1]);
```

This will throw `ArrayStoreException` (if the wrong type of object is stored in an array) in runtime.

If you wonder why Java Array is covariant, it's a decision made by language designers to allow a different kind of polymorphism back in the days when there were no Generics.

> Due to the `invariance` property of the `Generic` and the `covariance` property of the `Array` in Java, it is strongly recommended to prefer `List` over `Array` in Java code.
{: .prompt-info}

---

## Restrictions of Generics

Let's switch back to the practical aspect of Generics. Let's have a look at some restrictions on Generics.

### Type argument cannot be a primitive

It's easy to understand why, once you understand how `Type Erasure` works. As we already showed, the Java compiler is doing Type Erasure during compilation, which means it replaces Generic Types with `Object` (or some of its subclasses, as we'll show in the next part of the series).

Hence, primitive types could not be used as `type arguments`.

### Type parameters cannot be used in a static context

The easiest way to understand this statement is to go through an example:

```java
public class Inventory<T> {
    private static T inventoryType;
}
```

This code will fail to compile. Let's understand why is that. Say we would like to make 2 Inventory instances, one of `Packet` class and the other of `Letter` class:

```java
Inventory<Packet> packetInventory = new Inventory<>();
Inventory<Letter> letterInventory = new Inventory<>();
```

What should the static field of `Inventory` class be? Is it `Packet` or `Letter`?

And what if we didn't create an instance of `Inventory` class at all?

Now it should be obvious that the type parameter `T` could not be resolved correctly in the case of the static field.

For the same reason, type parameters cannot be used in static methods and static initializers.

### Method overloading with different type arguments

This is impossible due to Type Erasure:

```java
public List<String> get();
public List<Integer> get();
```

After type erasure, we would effectively get:

```java
public List get();
public List get();
```

And it's invalid to have two methods with exactly the same signature

### Method overloading with type parameter and Object type

This is invalid due to type erasure, as methods would have exactly the same signature:

```java
class GenericsExample <T> {
    public T get(T t){
        return t;
    }

    public Object get(Object t) {
        return t;
    }
}
```

You should be able by now to figure out why this won't compile 🙂.

### Method overloading with type parameter and concrete type

Lastly, one tricky example.

This class definition is valid since after type erasure we don't have a clash of method signatures

```java
class GenericsExample <T> {
    public T get(T t){
        return t;
    }

    public String get(String t) {
        return t;
    }

    public String getExtended(String t) {
        return t + " extended";
    }
}
```

If you instantiate this class with an `Integer` type argument, everything will be fine. Java compiler will effectively produce two get methods with `Object` and `String` as a parameter and return type. Also, every call to the `get` method of the `Object` type is explicitly type-casted with the `Integer` class. There is a clear distinction between calls to those methods.

But let's take a look at the case when you instantiate this class with `String` type argument.

Since type erasure will replace `T` with `Object` there is no method signatures collision as we've seen before, as we'll have:

```java
public Object get(Object t){
    return t;
}

public String get(String t) {
    return t;
}
```

And those method signatures are different.

But if there is an invocation of the `get` method in your code, a compiler will try to do explicit casting and at that point, it realizes that both `Object` and `String` versions of the `get` method would be eligible for this code.

So the compiler will give you a compilation error **exactly** in places where you use the `get` method as it is ambiguous.

Nonetheless, as we said you can instantiate this class with the `String` type argument and use other methods that are not ambiguous at the compile time, such as `getExtended` in our example.

To summarize:
- this class definition is valid
- instantiation with any type argument is valid, even if that type is seemingly causing method signature collision
- you can use all the methods from such `Generic` class, except for the one that causes method signature collision

> It is strongly recommended not to use such class definitions that causes compilation errors for certain type arguments only. At least you should provide distinct method names to mitigate the risk of falling into this kind of situations.
{: .prompt-danger}

---

[Generics in Java part 1 of 3]({% post_url 2022-09-25-generics-in-java-part-1-of-3 %})
