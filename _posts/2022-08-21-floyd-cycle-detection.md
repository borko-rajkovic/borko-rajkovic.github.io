---
author: borko
title: "Floyd cycle detection"
description: "To check for the existence of a cycle in Linked List, you can use Floyd cycle detection (tortoise and hare algorithm). I created a mini-game that shows the use of Floyd cycle detection, you can check it out here: https://tile-walker.web.app/"
date: 2022-08-21 10:10:00 +0200
categories: [algorithms]
tags: [algorithms,cycle detection]
image: /assets/img/posts/2022-08-21-floyd-cycle-detection/thumbnail.png
---

## TL;DR

To check for the existence of a cycle in Linked List, you can use Floyd cycle detection (tortoise and hare algorithm).

I created a mini-game **Tile walker** 🏃‍♂️ that shows the use of Floyd cycle detection, you can check it out <a href="https://tile-walker.web.app/" target="_blank">here</a>.

![Floyd cycle detection](/assets/img/posts/2022-08-21-floyd-cycle-detection/thumbnail_2.png)

---

## **Linked list** data structure

**Linked list** is one of the simplest data structures. The idea behind it is to make related objects connect by holding a pointer or a reference to the next object in a collection.

There are three main types of `Linked List`:

- **Simple Linked List** - each member contains information about the next member in the list (if it exists) with a `next` pointer/reference. The last element's `next` pointer points to `null`.
- **Doubly Linked List** - same as **Simple Linked List**, with additional `previous` pointer/reference that points to a previous member of the list. The first element's `previous` pointer points to `null`.
- **Circular Linked List** - same as **Doubly Linked List**, but `previous` pointer of the first element in the list points to the last element in the list, and the last element's `next` pointer points to the first element in the list, effectively connecting the list into a loop.

We are going to use a **Simple Linked List** for the problem we are going to present.

---

## Traversal of the Linked List

Let's take an example of a simple **Linked List** of integers:

![Simple Linked List](/assets/img/posts/2022-08-21-floyd-cycle-detection/Simple_Linked_List.png){: .shadow style="max-width: 280px; width: 90%" }
_Simple Linked List_

<!-- ```
╔════╦═══════╦═════════╗
║ id ║ value ║ next_id ║
╠════╬═══════╬═════════╣
║ 0  ║ 805   ║ 1       ║
╚════╩═══════╩═════════╝
╔════╦═══════╦═════════╗
║ id ║ value ║ next_id ║
╠════╬═══════╬═════════╣
║ 1  ║ 248   ║ 2       ║
╚════╩═══════╩═════════╝
╔════╦═══════╦═════════╗
║ id ║ value ║ next_id ║
╠════╬═══════╬═════════╣
║ 2  ║ 324   ║ 3       ║
╚════╩═══════╩═════════╝
╔════╦═══════╦═════════╗
║ id ║ value ║ next_id ║
╠════╬═══════╬═════════╣
║ 3  ║ 123   ║ 4       ║
╚════╩═══════╩═════════╝
╔════╦═══════╦═════════╗
║ id ║ value ║ next_id ║
╠════╬═══════╬═════════╣
║ 4  ║ 518   ║ null    ║
╚════╩═══════╩═════════╝
``` -->

Traversal is very simple. We start from the first member, take its value and go on to the next member following the pointer, until the pointer of the member points to null.

In this way we are getting the following result:

```
805, 248, 324, 123, 518
```

---

## Linked List cycle detection

Linked List is not always going exclusively forward. Sometimes you need to point to a member in a list that already exists.

For example:

![Linked List with a cycle](/assets/img/posts/2022-08-21-floyd-cycle-detection/Linked_List_with_a_cycle.png){: .shadow style="max-width: 280px; width: 90%" }
_Linked List with a cycle_


<!-- ```
╔════╦════════════════╦══════════════╗
║ id ║ step_name      ║ next_step_id ║
╠════╬════════════════╬══════════════╣
║ 0  ║ Start process  ║ 1            ║
╚════╩════════════════╩══════════════╝
╔════╦════════════════╦══════════════╗
║ id ║ step_name      ║ next_step_id ║
╠════╬════════════════╬══════════════╣
║ 1  ║ Wait for input ║ 2            ║  <---.
╚════╩════════════════╩══════════════╝      |
╔════╦════════════════╦══════════════╗      |
║ id ║ step_name      ║ next_step_id ║      |
╠════╬════════════════╬══════════════╣      |
║ 2  ║ Verify input   ║ 3            ║      |
╚════╩════════════════╩══════════════╝      |
╔════╦════════════════╦══════════════╗      |
║ id ║ step_name      ║ next_step_id ║      |
╠════╬════════════════╬══════════════╣      |
║ 3  ║ Process input  ║ 4            ║      |
╚════╩════════════════╩══════════════╝      |
╔════╦════════════════╦══════════════╗      |
║ id ║ step_name      ║ next_step_id ║      |
╠════╬════════════════╬══════════════╣ -----'
║ 4  ║ Notify user    ║ 1            ║
╚════╩════════════════╩══════════════╝
``` -->

As we can see in this example, this **Linked List** does not have an end member, as all members of the list point to some other member, and none of them points to `null`.

While traversing the list, you will repeatedly walk over the same list members:

```
Start process
Wait for input
Verify input
Process input
Notify user
Wait for input
Verify input
...
```

In this example members from id `1` to id `4` are connected in a loop, or in other words, they are forming a `cycle`.

Here one natural question arises: how can we detect the existence of a `cycle` in a **Linked List**?

The problem is formally known as `The cycle detection` problem.

---

## Naive approach

The easiest solution that can be implemented is to save pointers to a list of members as we traverse the Linked List. In every iteration, we are checking if the pointer to that list member is already present in our set of pointers.

Here is the naive approach for the example above:

| iteration | current set state | current id | element was in set |
| --------- | ----------------- | ---------- | ------------------ |
| 1         | []                | 0          | no                 |
| 2         | [0]               | 1          | no                 |
| 3         | [0, 1]            | 2          | no                 |
| 4         | [0, 1, 2]         | 3          | no                 |
| 5         | [0, 1, 2, 3]      | 4          | no                 |
| 6         | [0, 1, 2, 3, 4]   | 1          | yes                |

<!-- ```
╔═══════════╦═════════════════════╦═════════════╦════════════════════╗
║ iteration ║ current set state   ║ current id  ║ element was in set ║
╠═══════════╬═════════════════════╬═════════════╬════════════════════╣
║ 1         ║ []                  ║ 0           ║ no                 ║
╠═══════════╬═════════════════════╬═════════════╬════════════════════╣
║ 2         ║ [0]                 ║ 1           ║ no                 ║
╠═══════════╬═════════════════════╬═════════════╬════════════════════╣
║ 3         ║ [0, 1]              ║ 2           ║ no                 ║
╠═══════════╬═════════════════════╬═════════════╬════════════════════╣
║ 4         ║ [0, 1, 2]           ║ 3           ║ no                 ║
╠═══════════╬═════════════════════╬═════════════╬════════════════════╣
║ 5         ║ [0, 1, 2, 3]        ║ 4           ║ no                 ║
╠═══════════╬═════════════════════╬═════════════╬════════════════════╣
║ 6         ║ [0, 1, 2, 3, 4]     ║ 1           ║ yes                ║
╚═══════════╩═════════════════════╩═════════════╩════════════════════╝
``` -->

As we can see, we are expanding our set of pointers until we come across the pointer that is already present in the set, or we come across a null pointer, which means that list is cycle-free.

---

### Complexity analysis

Complexity analysis is very simple for this algorithm:

**Time complexity** is `O(N)`, because we need to traverse all list members and this is the low as we can get, so this is already the greatest time complexity.

**Space complexity** is `O(N)`, because in the worst case we need to store all list elements in the set until we get to the end of the list that does not have `cycle` in itself.

As we can see from the analysis, we can affect only **space complexity**.

---

## Floyd cycle detection

The algorithm that was introduced by **Robert Floyd** is using 2 pointers with which **Linked List** is traversed in `O(N)` time, but since we are using exactly 2 pointers, space complexity is going to be constant, or `O(1)`.

Let's see how it works.

Both pointers start from the beginning of the list. The first pointer advances **one step** per iteration, whereas the second pointer advances **two steps** per iteration. If the faster pointer comes across a `null` pointer, it means that `Linked List` does not have a cycle in it. Otherwise, these two pointers will meet at some moment and they will point to the same list member, hence proving that there is a cycle.

We can show how it works in an example from above:

<!-- ```
╔═══════════╦════════════════╦═════════════════╗
║ iteration ║ first pointer  ║ second pointer  ║
╠═══════════╬════════════════╬═════════════════╣
║ 1         ║ 0              ║ 0               ║
╠═══════════╬════════════════╬═════════════════╣
║ 2         ║ 1              ║ 2               ║
╠═══════════╬════════════════╬═════════════════╣
║ 3         ║ 2              ║ 4               ║
╠═══════════╬════════════════╬═════════════════╣
║ 4         ║ 3              ║ 2               ║
╠═══════════╬════════════════╬═════════════════╣
║ 5         ║ 4              ║ 4               ║
╚═══════════╩════════════════╩═════════════════╝
``` -->

| iteration | first pointer | second pointer |
| --------- | ------------- | -------------- |
| 1         | 0             | 0              |
| 2         | 1             | 2              |
| 3         | 2             | 4              |
| 4         | 3             | 2              |
| 5         | 4             | 4              |

Here you can see the demo animation:

![Floyd cycle detection iterations](/assets/img/posts/2022-08-21-floyd-cycle-detection/iterations.gif){: .shadow style="max-width: 400px; width: 90%" }
_Floyd cycle detection iterations_

<!-- ### Iteration 1

```
      ╔════╦════════════════╦══════════════╗
      ║ id ║ step_name      ║ next_step_id ║
p1 -> ╠════╬════════════════╬══════════════╣
p2 -> ║ 0  ║ Start process  ║ 1            ║
      ╚════╩════════════════╩══════════════╝
      ╔════╦════════════════╦══════════════╗
      ║ id ║ step_name      ║ next_step_id ║
      ╠════╬════════════════╬══════════════╣
      ║ 1  ║ Wait for input ║ 2            ║  <---.
      ╚════╩════════════════╩══════════════╝      |
      ╔════╦════════════════╦══════════════╗      |
      ║ id ║ step_name      ║ next_step_id ║      |
      ╠════╬════════════════╬══════════════╣      |
      ║ 2  ║ Verify input   ║ 3            ║      |
      ╚════╩════════════════╩══════════════╝      |
      ╔════╦════════════════╦══════════════╗      |
      ║ id ║ step_name      ║ next_step_id ║      |
      ╠════╬════════════════╬══════════════╣      |
      ║ 3  ║ Process input  ║ 4            ║      |
      ╚════╩════════════════╩══════════════╝      |
      ╔════╦════════════════╦══════════════╗      |
      ║ id ║ step_name      ║ next_step_id ║      |
      ╠════╬════════════════╬══════════════╣ -----'
      ║ 4  ║ Notify user    ║ 1            ║
      ╚════╩════════════════╩══════════════╝
```


### Iteration 2

```
      ╔════╦════════════════╦══════════════╗
      ║ id ║ step_name      ║ next_step_id ║
      ╠════╬════════════════╬══════════════╣
      ║ 0  ║ Start process  ║ 1            ║
      ╚════╩════════════════╩══════════════╝
      ╔════╦════════════════╦══════════════╗
      ║ id ║ step_name      ║ next_step_id ║
p1 -> ╠════╬════════════════╬══════════════╣
      ║ 1  ║ Wait for input ║ 2            ║  <---.
      ╚════╩════════════════╩══════════════╝      |
      ╔════╦════════════════╦══════════════╗      |
      ║ id ║ step_name      ║ next_step_id ║      |
p2 -> ╠════╬════════════════╬══════════════╣      |
      ║ 2  ║ Verify input   ║ 3            ║      |
      ╚════╩════════════════╩══════════════╝      |
      ╔════╦════════════════╦══════════════╗      |
      ║ id ║ step_name      ║ next_step_id ║      |
      ╠════╬════════════════╬══════════════╣      |
      ║ 3  ║ Process input  ║ 4            ║      |
      ╚════╩════════════════╩══════════════╝      |
      ╔════╦════════════════╦══════════════╗      |
      ║ id ║ step_name      ║ next_step_id ║      |
      ╠════╬════════════════╬══════════════╣ -----'
      ║ 4  ║ Notify user    ║ 1            ║
      ╚════╩════════════════╩══════════════╝
```

### Iteration 3

```
      ╔════╦════════════════╦══════════════╗
      ║ id ║ step_name      ║ next_step_id ║
      ╠════╬════════════════╬══════════════╣
      ║ 0  ║ Start process  ║ 1            ║
      ╚════╩════════════════╩══════════════╝
      ╔════╦════════════════╦══════════════╗
      ║ id ║ step_name      ║ next_step_id ║
      ╠════╬════════════════╬══════════════╣
      ║ 1  ║ Wait for input ║ 2            ║  <---.
      ╚════╩════════════════╩══════════════╝      |
      ╔════╦════════════════╦══════════════╗      |
      ║ id ║ step_name      ║ next_step_id ║      |
      ╠════╬════════════════╬══════════════╣      |
p1 -> ║ 2  ║ Verify input   ║ 3            ║      |
      ╚════╩════════════════╩══════════════╝      |
      ╔════╦════════════════╦══════════════╗      |
      ║ id ║ step_name      ║ next_step_id ║      |
      ╠════╬════════════════╬══════════════╣      |
      ║ 3  ║ Process input  ║ 4            ║      |
      ╚════╩════════════════╩══════════════╝      |
      ╔════╦════════════════╦══════════════╗      |
      ║ id ║ step_name      ║ next_step_id ║      |
p2 -> ╠════╬════════════════╬══════════════╣ -----'
      ║ 4  ║ Notify user    ║ 1            ║
      ╚════╩════════════════╩══════════════╝
```

### Iteration 4

```
      ╔════╦════════════════╦══════════════╗
      ║ id ║ step_name      ║ next_step_id ║
      ╠════╬════════════════╬══════════════╣
      ║ 0  ║ Start process  ║ 1            ║
      ╚════╩════════════════╩══════════════╝
      ╔════╦════════════════╦══════════════╗
      ║ id ║ step_name      ║ next_step_id ║
      ╠════╬════════════════╬══════════════╣
      ║ 1  ║ Wait for input ║ 2            ║  <---.
      ╚════╩════════════════╩══════════════╝      |
      ╔════╦════════════════╦══════════════╗      |
      ║ id ║ step_name      ║ next_step_id ║      |
p2 -> ╠════╬════════════════╬══════════════╣      |
      ║ 2  ║ Verify input   ║ 3            ║      |
      ╚════╩════════════════╩══════════════╝      |
      ╔════╦════════════════╦══════════════╗      |
      ║ id ║ step_name      ║ next_step_id ║      |
p1 -> ╠════╬════════════════╬══════════════╣      |
      ║ 3  ║ Process input  ║ 4            ║      |
      ╚════╩════════════════╩══════════════╝      |
      ╔════╦════════════════╦══════════════╗      |
      ║ id ║ step_name      ║ next_step_id ║      |
      ╠════╬════════════════╬══════════════╣ -----'
      ║ 4  ║ Notify user    ║ 1            ║
      ╚════╩════════════════╩══════════════╝
```

### Iteration 5

```
      ╔════╦════════════════╦══════════════╗
      ║ id ║ step_name      ║ next_step_id ║
      ╠════╬════════════════╬══════════════╣
      ║ 0  ║ Start process  ║ 1            ║
      ╚════╩════════════════╩══════════════╝
      ╔════╦════════════════╦══════════════╗
      ║ id ║ step_name      ║ next_step_id ║
      ╠════╬════════════════╬══════════════╣
      ║ 1  ║ Wait for input ║ 2            ║  <---.
      ╚════╩════════════════╩══════════════╝      |
      ╔════╦════════════════╦══════════════╗      |
      ║ id ║ step_name      ║ next_step_id ║      |
      ╠════╬════════════════╬══════════════╣      |
      ║ 2  ║ Verify input   ║ 3            ║      |
      ╚════╩════════════════╩══════════════╝      |
      ╔════╦════════════════╦══════════════╗      |
      ║ id ║ step_name      ║ next_step_id ║      |
      ╠════╬════════════════╬══════════════╣      |
      ║ 3  ║ Process input  ║ 4            ║      |
      ╚════╩════════════════╩══════════════╝      |
      ╔════╦════════════════╦══════════════╗      |
      ║ id ║ step_name      ║ next_step_id ║      |
p1 -> ╠════╬════════════════╬══════════════╣ -----'
p2 -> ║ 4  ║ Notify user    ║ 1            ║
      ╚════╩════════════════╩══════════════╝
``` -->

---

## Finding the beginning of the cycle

If there is a `cycle` in a LinkedList, we can find out the start of the cycle in the following way:

- reset one pointer to the start of the **Linked List**
- advance both pointers at the same speed (1 list member per iteration) until they meet
- their meeting point is the start of the `cycle`

That explanation is probably counter-intuitive, so let's try to explain what exactly happens here:

![Cycle start calculation](/assets/img/posts/2022-08-21-floyd-cycle-detection/cycle_start_calculation.png){: .shadow style="max-width: 500px; width: 90%" }
_Cycle start calculation_

> For simplicity, the formula we are going to derive here works only for cases where `X <= C`. Later we'll cover the general formula, you can check it [here](#what-if-x--c)
{: .prompt-warning }

Let's express the distance traveled by both slow and fast pointers before they meet each other:

`slow_pointer_distance = X + Y`

`fast_pointer_distance = (X + Y + Z) + Y = X + 2Y + Z`

We know that the slow pointer is exactly 2 times slower than the fast pointer. So, when they meet, the fast pointer must travel 2 times greater distance:

`fast_pointer_distance = 2 X slow_pointer_distance`

With the given equations above, we get:

`X + 2Y + Z = 2 (X + Y)`

`X + 2Y + Z = 2X + 2Y`

`X + Z = 2X`

`X = Z`

If we translate `X` and `Z` for what they really are, we get:

distance from `First node` to `Cycle start` == distance from `Pointer meeting point` to `Cycle start`

Now it should be clear that if we move one pointer to the first node and move pointers at the same speed (one step per iteration), they will meet at the Cycle start!

![Finding cycle start](/assets/img/posts/2022-08-21-floyd-cycle-detection/finding_cycle_start.gif){: .shadow style="max-width: 90%" }
_Finding Cycle start_

---

## Calculating length of the cycle

Once we know the starting point of the `cycle`, we can calculate its length in a very simple way.

All we need to do is to:

- start with both pointers from the start of the `cycle`
- traverse with another pointer until we meet the fixed pointer

Simple as that 😃

![Cycle length](/assets/img/posts/2022-08-21-floyd-cycle-detection/cycle_length.gif){: .shadow style="max-width: 90%" }
_Cycle length_

---

## What if X > C

In the example above we assumed that `X <= C`. In case that `X > C`, the formula would change a bit, since the `fast` pointer can make `N` number of cycle traversals until it finally meets the `slow` pointer, where `N > 1` (in general case `N` cannot be lower than `1` since faster pointer will go with `2X` speed the slow pointer, so it will for sure make at least one round trip of the cycle if the cycle exists).

The fast pointer would travel:

`X + N * C + Y`

And the slow pointer is covering the distance of:

`X + Y`

Since the fast pointer covers 2 times the distance of the slow pointer, we get the following equation:

`2 (X + Y) = X + N*C + Y`

`2X + 2Y = X + N*C + Y`

`X + Y = N*C`

`X = N*C - Y`

Rewrite `N*C` as `(N - 1) C + C`, and then `(N - 1) C + (Y + Z)`

`X = (N - 1) C + (Y + Z)- Y`

And finally, we get the general formula for all cases:

`X = (N - 1) C + Z`

We can notice that for `N == 1`, we get the same as before: `X = Z`

But what this really means is that distance `X` is the same as the number of cycles `N-1` that fast pointer travels + distance `Z`. So when advancing one pointer from the start of the list, and another from their meeting point, the second one will travel `N-1` cycles and then meet the first pointer at the start of the cycle.

It doesn't matter if you didn't quite catch all of this, here is the demo that can help you out 🙂

![When X > C](/assets/img/posts/2022-08-21-floyd-cycle-detection/when_x_greater_than_y_plus_z.gif){: .shadow style="max-width: 90%" }
_When X > C_

---

## When Cycle start node == Pointers Meeting point

There is a case where the `cycle start` node is the same node as the `pointers meeting point` node.

In such a case, we have:

`slow_pointer_distance = X`

`fast_pointer_distance = X + N*C`

Since `fast` pointer is covering `2X` distance of `slow` pointer, we have:

`2*X = X + N*C`

`X = N*C`

This means that distance from the start of the list to the start of the cycle is exactly the distance `fast` pointer traveled around the cycle 😃!

In other words, whenever `X` is an exact multiple of cycle length `C`, we know that the `pointers meeting point` and the `cycle start` nodes are the same node.

![When Meeting point is same as Cycle start](/assets/img/posts/2022-08-21-floyd-cycle-detection/when_meeting_point_is_same_as_cycle_start.png){: .shadow style="max-width: 90%" }
_When Meeting point is same as Cycle start_

---

## Tile Walker - mini game

Here is an educative mini-game **Tile Walker** that is using `Floyd cycle detection` to detect cycle in a **Linked List** in order to conclude that **Tile Walker** has exhausted all tiles it will walk over in a game.

![Floyd cycle detection game](/assets/img/posts/2022-08-21-floyd-cycle-detection/thumbnail_2.png)

You can check out the game [here](https://tile-walker.web.app/).

And code can be found [here](https://github.com/borko-rajkovic/tile-walker).

### Rules

The player is walking in the same direction until he comes across an obstacle. Then it will make a turn of 90 degrees and continue walking in that direction.

The goal is to set up the board, so the **Tile Walker** can cover as many tiles as possible.

### Where **Floyd cycle detection** is used here?

When a player starts the game, it will simulate how **Tile Walker** is going over the playfield. Once it's not able to cover more tiles, the game stops. To detect when this happens, we use **Floyd cycle detection**, but with a twist, since we need to compare not just the location of the player, but its direction as well.

For more info, please check out the repo [here](https://github.com/borko-rajkovic/tile-walker).
