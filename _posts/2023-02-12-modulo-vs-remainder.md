---
layout: post
title: Modulo vs Remainder
author: borko
description: Modulo and Remainder operators both produce the same result in cases
  where operands have the same sign. In this post you'll learn about the difference
  between Modulo and Remainder, why would you choose one over the other and how to
  calculate Modulo in languages that don't have built-in Modulo operator.
categories:
- algorithms
tags:
- algorithms
- modulo
- remainder
image: "/assets/img/posts/2023-02-12-modulo-vs-remainder/thumbnail.png"
date: 2023-02-12 22:48 +0100
---
Once I had to migrate an application from Python to Javascript. Even though the code was copied almost exactly, tests were failing. After some time I finally found out that the `%` operator is not the same in **Javascript** and **Python**. It's the **Modulo** operator for **Python**, but the **Remainder** for **Javascript**. Luckily, the fix is rather easy, you can find it at the end of this post.

## Quick refresh on `%` operator

Most tutorials for beginners in programming at some point teach to use the operator `%` to get the remainder after dividing one number by the other.

Examples:

```js
7 % 5 = 2
4 % 2 = 0
```

And you can verify it quickly:

- divide 7 by 5 `7 / 5 = 1.4`
- round the result (in this case to 1) and multiply by the divisor (5) `1 * 5  = 5`
- 7 - 5 = 2

With the `%` operator, you can check for any positive integer if it's divisible by another positive integer.

This is handy in checking if the number is odd/even (by checking if it's divisible by 2):

```js
const isEven = number % 2 === 0
```

## Why should I care?

The `%` is usually applied to positive integers. But the difference appears when considering negative values.

Let's see them in action.

**Javascript** uses `%` as the **Remainder** operator:


```js
console.log(7 % 5);
// output: 2

console.log(-7 % 5);
// output: -2

console.log(7 % -5);
// output: 2

console.log(-7 % -5);
// output: -2
```


**Python** uses `%` as the **Modulo** operator:

```py
print(7 % 5)
#  output: 2

print(-7 % 5)
#  output: 3

print(7 % -5)
#  output: -3

print(-7 % -5)
#  output: -2
```

## Definitions

For the operation

```
n % d
```

where:

- n - dividend
- d - divisor

Modulo/Remainder is calculated with the formula:

`n - d * q`

- **Remainder** uses the **truncate** of the result of the `n / d` to get quotient **q**
- **Modulo** uses the **floor** of the result of the `n / d` to get quotient **q**

When both operands are of the same sign, **floor** and **truncate** will produce the same result, otherwise, they can differ by at most 1:

```
floor(-1.5) = -2
truncate(-1.5) = -1
```

> There are other ways to calculate the quotient, but **truncated** and **floored** are the most used ones. For a complete list and to check what your programming language uses, check [this](https://en.wikipedia.org/wiki/Modulo#In_programming_languages) Wiki page.
{: .prompt-info}

### Calculation

`-7 % 5`

- the modulo is calculated as:

```
-7 - (5 * (floor(-7/5)))=
-7 -(5*(-2))=
-7 + 10 =
3
```

- the remainder is calculated as 

```
-7 - 5 * (truncate(-7/5)) = 
-7 -(5*(-1)) = 
-7 + 5 = 
-2
```

## Example of Remainder

The remainder is useful to check if a number is divisible by another number.

Let's say that you have a list that represents days in a week (Monday to Sunday), going from 0 to 6:

`[0 1 2 3 4 5 6]`

If the current day is **Friday** (4), how would you calculate what day would it be after 18 days?

You could use either **modulo** or **remainder** like this:

`(4 + 18) % 7 = 1`

So, the result is **Tuesday** (2).

## Example of Modulo

That's great, but what if you need to calculate what day it was 20 days ago?

In this case, you could use the **modulo** to find out that:

`(4 - 20) mod 7 = 5`

And the result is **Saturday** (6).

## Intuition for remainder and modulo

For the non-negative numbers, you can see how the numbers wrap around the range 0 - 6:

```
n
0   1   2   3   4   5   6   7   8   9

n % 7
0   1   2   3   4   5   6   0   1   2
```

But if we include negative numbers, we'll get:

```
n
-9  -8  -7  -6  -5  -4  -3  -2  -1  0   1   2   3   4   5   6   7   8   9

n rem 7
-2  -1  -0  -6  -5  -4  -3  -2  -1  0   1   2   3   4   5   6   0   1   2

n mod 7
5   6   0   1   2   3   4   5   6   0   1   2   3   4   5   6   0   1   2
```

For the negative values you can see that the **remainder** is producing the negative values that repeat once they reach the limit (remember **d** in `n % d`),

Where **modulo** is repeating the same sequence we had in a positive spectrum (`[0,1,2,3,4,5,6]` in our case).

## Convert from Remainder to Modulo

The fix I mentioned at the beginning of the post is actually quite a simple one.

Here is the function that will calculate the **modulo** of numbers **a** and **b**:

```js
function modulo(a, b) {
    return ((a % b) + b) % b;
}
```
