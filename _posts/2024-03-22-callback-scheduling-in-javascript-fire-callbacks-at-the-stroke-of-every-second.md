---
layout: post
title: 'Callback Scheduling in JavaScript: Fire Callbacks at the
  Stroke of Every Second'
date: 2024-03-22 20:41 +0100
author: borko
description: Check out how to schedule callbacks to fire on the stroke of every second. Also, be aware of the drifting away phenomena and check how to fix this problem easily.
categories:
- javascript
tags:
- javascript
image: "/assets/img/posts/2024-03-22-javascript-second-scheduling/thumbnail.png"
---
Sometimes you would like to execute a callback exactly on the stroke of each second. There are many cases when you would like to achieve such behavior, here are just a few examples:

- New Year's Countdown
- Birthday Countdown
- Precise alarm clock
- Automating HTTP requests
- Precise animation triggers

You may think it's not a big deal if you miss the exact stroke of a second, but it would be really nice to achieve precision when it comes to triggering animation at the exact time (like New Year's Countdown).

## Logging time

First, let's define our simple callback function `logTime` for testing:

```js
const logTime = () => console.log(+new Date());
```

Converting Date to number will give us time since January 1. 1970 UTC, **including milliseconds**.

That is important, as we'll see how much deviation we get from the stroke of a second.


## First attempt: setInterval

Whenever you need to achieve some periodic execution in Javascript, what first comes to mind is the `setInterval` [Web API](https://developer.mozilla.org/en-US/docs/Web/API/setInterval).

In short, the `setInterval` function takes a callback as the first parameter and delay as the second.

```js
setInterval(logTime, 1000);
```

This will schedule the execution of the `logTime` function each second, but the first execution will not have to be invocated at the stroke of a second (and it's a rare chance it is). Here is the result of the `setInterval` call:

```
1711139891173
1711139892173
1711139893173
1711139894173
```

## Fixing the first invocation

You don't want to start interval on some arbitrary point in time, but rather on the stroke of a second.

So, essentially what you want to do is to **delay** setting up an interval until the next stroke of the second.

What is used for delaying invocations in Javascript? That's right, the `setTimeout` Web API.

And how much time would you need to delay?

That should be easy: as much time as we're left with until the next stroke of the second.

We can get milliseconds of the current time using:

`new Date().getMilliseconds()`

Since there are 1000 milliseconds in the second, we need to subtract current milliseconds from 1000 to get to the next stroke of a second.

Let's try it out.

Similar to `setInterval`, the `setTimeout` will take the callback as the first parameter and delay as the second.

```js
setTimeout(logTime, 1000 - new Date().getMilliseconds());
```

The result is probably what you expected, the method was executed at the stroke of the second:

```
1711144065000
```

## Tying up the first invocation with interval

Now that we fixed the first invocation, it should be straightforward to tie it up with an interval:

```js
setTimeout(
  () => setInterval(logTime, 1000),
  1000 - new Date().getMilliseconds()
);
```

Finally, we got an interval that is executing a callback on each stroke of a second:

```
1711144386000
1711144387000
1711144388000
1711144389000
```

## Drifting away

Javascript `setInterval` and `setTimeout` Web APIs don't guarantee that a callback will be executed at exactly the requested time.

Furthermore, `setInterval` is susceptible to **drifting away**, especially if run for a long time.

In order to fix this, we need to re-synchronize time intervals, which should keep us safe from **drifting away** phenomena.

## Fixing drift-away phenomena

Instead of using `setInterval`, we can use `setTimeout` in a recursive function. After every invocation, we'll calculate the next invocation time and schedule another execution using `setTimeout`:

```js
const clockTick = (callback) =>
  setTimeout(() => 
    {
      callback();
      clockTick(callback);
    }
    ,1000 - new Date().getMilliseconds()
  );

clockTick(logTime);
```

This should give us a similar result as before, but with a stronger guarantee of the precise execution:

```
1711146244000
1711146245000
1711146246000
1711146247000
```

## Long execution times

In general, you should avoid running expensive computations in callbacks that are scheduled. Depending on the desired behavior you can choose between two options:

- Block scheduling new execution until the current one is finished (this approach will skip over some invocations if they are not able to be scheduled on time)
- Allow multiple executions simultaneously

Our solution that we implemented so far allows for multiple executions, given that callback is an async function.

To convert it to block scheduling all we need to change is make the `clockTick` function async and **await** the callback to finish:

```js
const clockTick = (callback) =>
  setTimeout(async () => 
    {
      await callback();
      clockTick(callback);
    }
    ,1000 - new Date().getMilliseconds()
  );
```
