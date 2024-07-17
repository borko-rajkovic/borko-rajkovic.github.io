---
layout: post
title: Echoing Variables in Linux with newlines
date: 2024-07-20 15:22 +0200
author: borko
description: "Learn how to properly echo variable in Linux to keep newlines"
categories: [bash]
tags: bash
image: "/assets/img/posts/2024-07-20-echoing-variables-in-linux-with-newlines/thumbnail.png"
---

In Linux, a common pattern is to pipe data from one program to the other using pipe operator `|`. Sometimes your script grows large and then you save intermediary results to variables to better handle processing logic.

![Echoing variables in Linux while keeping newlines](/assets/img/posts/2024-07-20-echoing-variables-in-linux-with-newlines/thumbnail.png){: .shadow style="max-width: 90%" }

Depending on the terminal you use, it can happen that once you **echo** a variable it loses empty lines:

```sh
var="line1
line2
line3
"

echo $var
# line1 line2 line3
```

## Quick Answer

Just wrap your variable in a double-quoted string!


```sh
var="line1
line2
line3
"

echo "$var"
# line1
# line2
# line3
```

## Why does it happen?

If a variable is used without double quotes, they are expanding in the following way:

- using IFS content is split into separate words (here newlines are usually lost due to IFS defaults, check note)

> The Internal Field Separator (IFS) is a special shell variable in Bash that determines how words are split when the shell interprets input. By default, IFS is set to a space, tab, and newline.
{: .prompt-info}

- each of the words will be substituted with pathname expansion, where wildcards such as * are expanding to files that match the condition

> Wildcards that are supported for path expansion:
> 
> \* - Matches any string, including the null string
> 
> ? - Matches any single (one) character.
> 
> [...] - Matches any one of the enclosed characters.
{: .prompt-info}

- only then all the results are passed to a command such as the **echo** for printing on a terminal


## When path expansion does matter

Let's say that you have some comment as the value of your variable and you would like to **echo** it out:

```sh
var="/* Copyright 2024 */"

echo $var
# /0 /bin /boot /cdrom /dev /etc /home /lib /lib32 /lib64 /libx32 /lost+found /media /mnt /opt /proc /root /run /sbin /snap /srv /swapfile /sys /tmp /usr /var Copyright 2024 bin/ boot/ cdrom/ dev/ etc/ home/ lib/ lib32/ lib64/ libx32/ lost+found/ media/ mnt/ opt/ proc/ root/ run/ sbin/ snap/ srv/ sys/ tmp/ usr/ var/
```

Now it's obvious that not only it's unpredictable, but also dangerous to expand variables for arbitrary Linux commands/programs.

## How double quotes are helping?

When the variable is put inside double quotes it will be substituted for its exact value.

Since it's part of the string, there are no substitutions and path expansions.

```sh
var="/* Copyright 2024 */"

echo "$var"
# /* Copyright 2024 */
```

> In case you expect anything more than a single word that doesn't have special characters, you should enclose variable in double quotes when evaluating its content
{: .prompt-warning}
