---
layout: post
title: Generics in Java part 3 of 3
author: borko
description: Generics in Java had been introduced since Java 5. We'll see how they contribute to code reusability by adding an extra layer of abstraction over reference types. In this part, we are going to introduce bounded type parameters, generic methods and wildcards for generics.
categories:
- java
tags:
- java
- generics
image: "/assets/img/posts/2022-10-16-generics-in-java-part-3-of-3/thumbnail.png"
date: 2022-10-16 18:07 +0200
---

![thumbnail](/assets/img/posts/2022-10-16-generics-in-java-part-3-of-3/thumbnail.png)

This article is the third and final part of the series `Generics in Java`.

Here are links to all parts:

- [Generics in Java part 1 of 3]({% post_url 2022-09-25-generics-in-java-part-1-of-3 %})
- [Generics in Java part 2 of 3]({% post_url 2022-10-05-generics-in-java-part-2-of-3 %})
- [Generics in Java part 3 of 3]({% post_url 2022-10-16-generics-in-java-part-3-of-3 %}) 

Please check out parts 1 and 2 so you can follow along easily, as we cover the basic syntax and concepts such as type erasure and invariance.

In this part, we are going to introduce bounded type parameters, generic methods and wildcards for generics.

## Bounded type parameters

We've seen the power of Generics when it comes to making some class/interface generic without losing type safety. Now as great as it is, there still might be a place for improvement.

Let's say you want to restrict users of your `Generic class` to certain types (that have something in common). For example, what if you want to use subtypes of the `List` interface as your `type parameter`? Explicit type casting is one solution, but then we lose type safety.

We know that classes and interfaces can be subtyped using inheritance. Generics offers you a similar mechanism to restrict type parameters in such a way that the type argument must be either the same class or its subtype.

Such parameters are called `Bounded type parameters`.

Here is an example:

```java
class GenericsClass <T extends SomeBaseType>
```

Now, the type argument can be either `SomeBaseType` or any of its subtypes (interface or class).

One classical example of this usage is using a Java Collections Framework:

```java
class GenericsExample <T extends List>
```

This class could be instantiated in the following ways:

```java
GenericsExample<List> listExample = new GenericsExample<>();
GenericsExample<ArrayList> arrayListExample = new GenericsExample<>();
GenericsExample<LinkedList> linkedListExample = new GenericsExample<>();
```

When we try to use the type that is not of the bounded type or its subtype, we get a compilation error:

```java
GenericsExample<Collection> collectionExample = new GenericsExample<>();
```

Is there any other benefit to this bounded type parameter? Yes, as now the generic class can make sure that certain methods that come from bounded type are now available. In this example, since we know that type `T` is a `List` or its subtype, we can be sure that it must implement methods of the `List` interface.

In other words, our `Generic class` can access methods that are defined by the bounded type.

Therefore, we can do something like this:

```java
class GenericsExample <T extends List> {
    private T something;
    
    public Integer getSizeOfSomething() {
        return something.size(); 
    }
}
```

If you remember `Type Erasure`, we said that the `type arguments` are replaced by the `Object` class. Well, in the case of a `bounded type parameter`, the `type argument` is replaced by the bound itself.

> This happens due to `inheritance` being `covariant` in Java. Check out part 2 of this series for more information.
{: .prompt-info}

In the last example, it would be `List`. Of course, if we extended the class with the `List<Integer>` for example, it would still be the `List`, since there are no generic types in runtime (but at the compile time, Java will make sure you cannot put the item of the `String` type into the `List<Integer`). Keep in mind that even at the `bounded type parameters` you should prefer the usage of full generic type instead of `Raw Type`.

After compilation, our `GenericsExample` class would look something like this:

```java
class GenericsExample {
    private List something;
    
    public Integer getSizeOfSomething() {
        return something.size(); 
    }
}
```

If you have multiple type parameters, you can make bounds on the previously set `type parameter`. For example, we can extend the second type parameter with the first type parameter:

```java
    class GenericsExample <T1 extends List<String>, T2 extends T1>
```

And both of them will be replaced with `List` after `type erasure` in the compilation.

### Valid bound types

Here is the list of the valid bound types:

- Class
- Interface
- Enum
- Parameterized type

> We can only use the `extends` keyword with bounds for type parameters, there is no `implements` keyword in this context.
{: .prompt-warning}

It should be clear by now how to use `class`, `interface` and `enum` as bounds. We already have shown an example of `List<String>` bound. But what if we don't want to hardcode the type argument for this bound? We can use `parameterized type` bound:

```java
<T extends Comparable<T>>
```

In this example, we can instantiate a class with any type `T` that implements a `Comparable` interface of the same type `T`.

For example:

```java
class GenericsExample<T extends Comparable<T>> {
...
}

GenericsExample<Integer> genericsExample = new GenericsExample<>();
```

And that is because the `Integer` class indeed has in its signature: `implements Comparable<Integer>` (effectively allowing the `Integer` type to be referred to with the `Comparable` type).

### Multiple bound type parameters

We can make `bounded type parameters` with not just one, but multiple types. Let's check an example:

```java
class GenericsExample <T extends List & Closeable> {}
```

In this example, we can see that `GenericsExample` is bounded by both `List` and `Closeable` interfaces.

The type argument must be a subtype of all bounds, not just one of them. In other words, it's a product algebraic data type.

Example:

```java
class GenericsDemo <T extends List & Serializable> {}
```

Valid:

```java
GenericsDemo<ArrayList> test = new GenericsDemo<>();
```

Invalid:

```java
GenericsDemo<List> test = new GenericsDemo<>();
```

There are some rules to multiple bounds on type parameters:
- If a class is one of the bounds, it must be specified first
- There can be just one class in the bounds
- For final classes and enums - the type argument is bound itself (because the final class does not have a subclass and the enum is essentially a final class)

### Invalid bound types

- primitive
- arrays

These should not be a surprise. Primitive cannot be used anywhere in `Generic` anyway, and the array is a `covariant` type transformation, which clashes with `Generic` which is an `invariant` type transformation. Thus, it is forbidden to use an array as a bounded type parameter to keep `invariance` property and make code much more type-safe (for `variance` check out part 2 of the series).

---

## Generic Methods

Java Generics allows you to have `Generic methods` in your classes. This is especially useful for static utility methods, as we'll see later.

The syntax for the `Generic methods` is the following:

```java
<T1, T2, ...> returnType methodName(T1 p1, T2 p2){}
```

Here, `<T1, T2, ...>` must be placed between the return type and modifiers, for example:

```java
private <T1, T2> void genericMethod(T1 p1, T2 p2){}
public static <T1, T2> void staticGenericMethod(T1 p1, T2 p2){}
```

> It's not required for the class to be `Generic` to have `Generic` methods in it. This also stands true for `Generic constructors`.
{: .prompt-info}

You can find a lot of `Generic methods` in the `Java Collections` framework. Let's now take a look at a few examples:

```java
<T> T[] toArray(T[] a); // Java.util.Collection
public static <T> boolean replaceAll(List<T> list, T oldVal, T newVal) {...} // Java.util.Collections
```

### Generic method - type scope

The `type parameters` of the `Generic methods` are completely independent of the class-level `type parameters`.

```java
class GenericsDemo<T> {
    <T> void go(T object){}
}
```

It's important to notice here that class-level parameter `T` is not the same as the method-level parameter `T`. When this method is used, it will take method-level parameter `T` instead of class-level one.

> The compiler would probably give you a warning like: "Type parameter 'T' hides type parameter 'T'"
{: .prompt-warning}

So, it's a good idea to avoid using the same `type parameter` name on the class level and generic method level.

On the other hand, we can use both class-level and method-level `type parameters` in one method:

```java
class GenericsDemo<E> {
    <T> void go(T obj1, E obj2){}
}
```

As you probably guess already, such a method can't be static, since we know we cannot use class-level `type parameters` in a static context.

### Bounded type parameters in generic methods

We can also use bounded type parameters:

```java
<T extends List & Serializable> void go(T object){}
```

Everything we discussed so far about the `bounded type parameters` stands true for method-level `type parameters`.

You can also have the method-level `type parameter` that extends the class-level `type parameter` like this:

```java
class GenericsDemo<E> {
    <T extends E> void go(T object){}
}
```

In the following example, we have the `type parameter` that extends the first `type parameter`:

```java
<T1, T2 extends T1> void go(T1 obj1, T2, obj2){}
```

### Method Invocation

We can either explicitly specify the `type argument`, or let the compiler infer the type from the method invocation.

```java
<T> T go(T object) {
    return object;
}

Double val1 = go(1.0);
String val2 = go("String");
```

In this example type argument for `val1` is inferred from the method itself and is of `Double` type (primitive type such as `double` is boxed into reference type `Double`).

For the `val2` type, the argument is `String`.

Type argument inference is used not only here, but in diamond notation, as we already have seen in previous chapters.

If the `type parameter` for the `Generic method` appears only in the return type of the method, then it is inferred from the calling context:

```java
<T> List<T> emptyList();
```

(this method is from the Collections class)

```java
List<String> list = Collections.emptyList();
```

In this example, calling context is helping out Java to determine the type argument for the method `emptyList()`, which is a `String`. On the left-hand side (`List<String>`) is what is called a `target type` and the compiler is going to pick it as a `type argument`.

Notice that even if the target type is `Collection<String>` compiler is smart enough to understand that `List` is a subtype of a `Collection`, so it will infer the `type argument` successfully.

Target type infernal works only if there are no parameter types in a method. In other words, Java will first try to figure out the `type argument` from the method parameters before looking at the return type. For example:

```java
<T> T go(T object) {
    return object;
}

Double val = go("java");
```

The `type argument` inferred here is `String` and it will throw the compilation error.

Inferral will get the most specific common super-type.

Example:

```java
<T> T go(T object, T object2) {
    return object;
}

var s = go("d", new ArrayList<String>());
```

In this case, both arguments are `Serializable` and that is what is inferred as the `type argument` for the variable `s` in this case.

In some rare cases, there can be a need for explicitly setting type arguments.

```java
class GenericsDemo {
    <T> T go(T object) {
        return object;
    }

    GenericsDemo gd = new GenericsDemo();
    var result = gd.<Number>go(1);
}
```

In this example, without using `.<Number>`, the variable `result` would be of the `Integer` type. Formally, this `type argument` is called `Type witness`.

If the method is called within the same class, it should be called using `this` keyword:

```java
    this.<Number>go(1);
```

For static methods it should be called with a class reference, even if a method is called in the same class:

```java
GenericsDemo.<Number>go(1);
```

---

## Wildcards

Let's refresh the terminology once more:

- The **Generic type** is a type with `type parameters` (definition of classes/interfaces)
- The **Parameterized type** on the other hand is the one that we get once we change type parameters with actual types, in other words, `type arguments`

Up to this point, all we had for `parameterized type` is the concrete type.

We have seen how we can make `type parameters` flexible by using `bounded type parameters`. This can be done in the definition of `generic types` (interfaces and classes), as well as `generic methods`.

Is there some way to make `parameterized types` more flexible?

Yes! Using something called `wildcards` for `type arguments`.

## Unbounded wildcards

We know that a Generic type through its `type parameter` indicates that it can be instantiated with some desired type.

For example:

```java
class Store<T> {}

void foo(Store<String> stringStore) {}
void bar(Store<Integer> intStore) {}
```

But a `parameterized type`, like a Generic type, can also indicate that its type argument can be of an `unknown` type. And that is done using the wildcard symbol - `?`.

Let's explain this with an example. If you have a method that is expecting some `List` and it's only going to use the methods of the `List` interface on its input parameter, then this method does not need to know about the type of elements that are present in a list. So, you could use `Raw type`:

```java
void methodThatTakesList(List input) {...}
```

But we already said this should be avoided since we lose type safety throughout our code. The solution is the usage of the `unbounded wildcard`:

```java
void methodThatTakesList(List<?> input) {...}
```

In this way, we don't hard code the type of elements in the list, but still keep type safety when using wildcard - `?` in the same way, as if we would use concrete type.

```java
void go(Store<?> someStore);
```

This can be read as `A Store of some type`. Method `go` doesn't know or doesn't care which kind of store it uses here.

This wildcard - `?` is known as an `unbounded wildcard`.

The key point here is to remember that the `type parameter` is part of the class definition and the `type argument` is the actual type we are substituting when using the Generic class/interface.

So `?` cannot appear in the class definition, only in its usage. In other words, the `Wildcard` can be used only as a `type argument`, it cannot be used as a `type parameter`.

Let's see why is that so:

```java
class Test<?,?> {
    private ? a;
}
```

This would not compile, as it's ambiguous.

The wildcard is often used as a type of `method parameter`, like in a method `go`. It can be used in assignments as well, but that is something that is not commonly seen.

### Unbounded wildcard vs Object

What is the difference between `Store<Object>` and `Store<?>`?

Because of invariance, to `Store<Object>` we can only assign `Store<Object>`:

```java
Store<Object> someStore = new Store<Object>();
```

But with wildcard we can assign an instance of any type:

```java
Store<?> someStore = new Store<String>();
Store<?> someStore = new Store<Integer>();
```

### Invocation of the class-level type methods

Take a look at the following example:

```java
int getCommonElementsCount(List<?> list1, List<?> list2) {
    int count = 0;
    for (Object element: list1) {
        if (list2.contains(element)){
            count++;
        }
    }

    return count;
}
```

Restriction on unbounded wildcards is that it's not possible to invoke methods that use class-level type parameters with any arguments except `null`.

Let's see that in action with the example of the method `getCommonElementsCount` we just showed.

Invalid

```java
list2.add(25);
```

Valid

```java
list2.add(null);
```

That is because we cannot make assumptions about the type of objects in the `getCommonElementsCount` method.

---

## Bounded wildcards

Bounded wildcards are mostly used in libraries, to make API flexible. Still, it's good to know how they work, as it can help you a lot if you stumble upon some library code while debugging, or if you need to make your Java library.

We can refer to parameterized types that use wildcards simply as wildcards types.

### Motivation for bounded wildcards

Generics are invariant, which gives us type safety. Because of that, it is very restrictive and sometimes we need more flexibility. For example, if we know that we wouldn't change the list, but just consume items from a list:

```java
void display(List<Animal> items)
```

Here we cannot pass `List<Cat>` or `List<Dog>` to this method due to the `invariance` of Generics.

One idea to solve this issue is to make separate methods:

```java
void display(List<Animal> items)
void display(List<Cat> items)
void display(List<Dog> items)
```

Not only this introduces the additional complexity of maintaining multiple methods that have the same purpose but is also illegal due to type erasure as we've already seen.

Using different method names with `raw types` is not recommended either as we'll lose type safety.

### Upper-bounded wildcard

Let's take a look at the example of the `upper-bounded` wildcard:

```java
void display(List<? extends Animal> items) {...}
```

Now `display` method can be invoked with a `List` of `Animal` items, or any subtype of `Animal`.

The second alternative is to use the `Generic method` with the `Bounded Type Parameter`:

```java
<T extends Animal> void display(List<T> items){...}
```

You may ask when to use one or the other since both of these approaches are perfectly valid.

The general rule of thumb is:

- if the `type parameter` is reused throughout the method as return type and parameter type, then we would go for the `generic method`
- if the `type parameter` is used only once, for example for a `parameter type`, we can use a `wildcard`

### Lower-bounded wildcard

Let's take a look at this example (assume that `Animal` is not an `abstract` class):

```java
void aggregate(List<Animal> list) {
    list.add(new Animal());
    list.add(new Cat());
    list.add(new Dog());
}

aggregate(new ArrayList<Animal>());
```

Due to invariance, we cannot use any type other than `Animal` as a `type argument` for the **list** parameter.

So, we cannot do something like this (assume that `LivingBeing` is the `super-type` of the `Animal` type):

```java
aggregate(new ArrayList<LivingBeing>());
aggregate(new ArrayList<Object>());
```

Due to invariance, even `super-type` of the `type argument` cannot be passed in here. We know that the `Object` is the `super-type` for all reference types.

`Upper-bounded wildcard` cannot help us here, because it will make sure that only that type and its subtypes are eligible for replacement. But we would like to allow `super-type` in this method parameter.

That is why for this purpose language designers introduced a `lower-bounded wildcard`:

```java
void aggregate(List<? super Animal> list) {...}
```

With this, we can now invoke the method with either that type or its `super-types`. In our case, that would be `Object`, `LivingBeing` and `Animal`.

Can you see that the `unbounded wildcard` is the same as the `lower-bounded wildcard` with an `Object` as its bound?

In other words, `<?>` is the same as `<? super Object>`.

We already said that you cannot invoke methods that use class-level type parameters with any arguments except null when using upper-bounded wildcards.

But with a `lower-bounded` wildcard, we can invoke methods of class-level type parameters only if the method is of a `lower-bounded` type or one of its super-types:

`List<? super Animal>`

```java
void aggregate(List<? super Animal> list){
    list.add(new Animal());
    list.add(new Cat());
    list.add(new Dog());
}

aggregate(new ArrayList<Animal>());
aggregate(new ArrayList<Object>());
```

In this example, `add()` method is defined on the lower-bounded wildcard, so we can use it and keep type safety here.

## Restrictions of wildcards

Wildcard can only have a single upper or lower bound!

This is illegal:

```java
<? extends bound1 & bound2>
```

## When to use which wildcard

If `parameterized type` is used to produce data, use upper bound:

```java
<? extends bound>
```

If `parameterized type` is used to consume data, use lower bound:

```java
<? super bound>
```

If `parameterized type` is used to produce data and doesn't care about the `type argument` then use the `unbounded wildcard` (the same as the `upper bound wildcard` with an `Object` as the bound):

```java
<?>
```

To summarize:

- If the method is used to produce data, we would use the `upper bound type parameter`
- If the method is used to consume data, we would use the `lower bound type parameter`
- If parameterized type is used as both producer and consumer, then use exact match

We cannot have upper-bound and lower-bound at the same type for the `type parameter`.

It doesn't make sense either to do it, since in one case you can use types' `super-type` and in the other `sub-type`. The intersection between class `subtypes` and `supertypes` is exactly that type!

### PECS

There is a mnemonic that you can use to remember this:

PECS -> Producer->`extends`, Consumer->`super`

## Examples

Examples from `Java Collections` framework:

```java
<T> boolean addAll(Collection <? super T> c, T... elements) // consuming elements, hence super
<T> void copy(List<? super T> dest, List<? extends T> src) // src is producer, dest is consumer
boolean disjoint(Collection<?> c1, Collection<?> c2) // we don't care about exact types of c1 and c2
<T> boolean replaceAll(List<T> list, T oldVal, T newVal) // types of list, oldVal and newVal must be the same
```

`addAll` is used to consume elements, that's why it's appropriate to use `super` here, as we want to be able to put the element of type `T` in a `Collection` of type `T` or its `supertype`:

```java
// Using Collection of Integer
Collection<Integer> intList = new ArrayList<>(List.of(1, 2, 3));
addAll(intList, 4, 5, 6);
// intList = [1, 2, 3, 4, 5, 6]

// Using Collection of Number (Number is the super-type of Integer)
Collection<Number> numberList = new ArrayList<>();
addAll(numberList, 4, 5, 6);
// numberList = [4, 5, 6]
```

`copy` is a great example of having both the consumer and the producer in the same method as parameters. Let's break this up, so we understand what is the desired behaviour of this method: 

We want to be able to add elements of type `T` or its `sub-type` to a list of `T` or its `super-type`

```java
List<Number> dest = new ArrayList<>(List.of(0, 0 , 0));

List<Integer> intSrc = new ArrayList<>(List.of(1, 2, 3));
List<Long> longSrc = new ArrayList<>(List.of(4L, 5L, 6L));
List<Double> doubleSrc = new ArrayList<>(List.of(7.0D, 8.0D, 9.0D));

copy(dest, intSrc); //      dest = [Integer 1, Integer 2, Integer 3]
copy(dest, longSrc); //     dest = [Long 4, Long 5, Long 6]
copy(dest, doubleSrc); //   dest = [Double 7.0, Double 8.0, Double 9.0]
```

`disjoint` will simply try to check if two sets are disjoint. For two sets to be disjoint, they can be of different types, and we don't care about concrete types of collections as we are going to compare values using the `equals()` method. But it's useful to keep type safety, as it's good practice (remember to avoid the use of the `raw types`).

Following the earlier example, we can do:

```java
boolean setsAreDisjoint = disjoint(dest, intSrc);
```

Finally, `replaceAll` is going to take a list and two values, so they all must be of the same type. Therefore, here we're going to use the `type parameter` itself instead of the wildcard:

```java
replaceAll(intSrc, 1, 4);
```
