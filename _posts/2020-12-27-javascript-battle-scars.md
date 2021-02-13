---
layout:     post
title:      "JavaScript: Battle Scars"
subtitle:   "Learn from my mistakes!"
date:       2020-12-27 12:00:00
author:     "Tomy Jaya"
header-img: "img/post-bg-js.jpeg"
tags:
- javascript
---


Just like many others, I used to gloss over details about JavaScript and just assumed it will behave like Java. 

This lazy habit turnt out to be fine most of the times; however, there were times, it bit me hard. 

The below are some of the battle scars I've learned the hard way: 

## 1. Returning `long` (64-bit) IDs in API

Take a look at this seemingly innocuous API response shape: 

```java
class Trade {
    long tradeId;
    String tradeType;
    long amount;
    String currency;
    // ... 
}
```

actual JSON: 

```json
{
  "tradeId": 793548328091516918,
  "tradeType": "BUY",
  "amount": 10000,
  "currency": "SGD"
}
```

Somewhere in your front-end code, you will consume this by using either `JSON.parse` or `fetch`'s `response.json()`. And you'll find out that the **tradeId** is truncated! (i.e. the last 2 digits became 00)

```javascript
{
  tradeId: 793548328091516900, // TRUNCATED!!! 
  tradeType: "BUY",
  amount: 10000,
  currency: "SGD"
}
```

Whoops! I knew JavaScript doesn't distinguish between floating point and integral numbers, what I didn't know was how much of its `Number` storage is reserved for the integral part. It turns out here's the allocation:

```
64 bit = 1 bit (sign) + 11 bits (exponent) + 52 bits (fraction)
```

`793548328091516918` is order of magnitudes greater than `2^52`. Hence, the truncation. 

As a post-mortem, we should have used `String` instead of `Long` for the things like transactionId which doesn't need any arithmetic operations to be performed on it. 

Further Readings: [How numbers are encoded in JavaScript](https://2ality.com/2012/04/number-encoding.html)

## 2. Using `parseInt` in `map`

When functional programming finally came to Java via Streams & Lambda, we all jumped to the bandwagon to write beautiful, functional, side-effect-free code. Collection-style `map` was all the craze. 

JavaScript also lended itself nicely to functional style, with arrays having built-in `map` and `functions` as first-class objects. However, this perfectly legitimate construct in Java:

```java
List.of("1","2","3")
    .stream()
    .map(Integer::parseInt) // method reference
    .collect(Collectors.toList());
```

when written in JavaScript in point-free style:

```javascript
['1','2','3'].map(parseInt); // [1, NaN, NaN]
```

resulted in an unexpected behavior. Why? It's because JavaScript `map` passes **not** only the value, but also the index of the item as well as the array being traversed into the mapping function. Incidentally, `parseInt` accepts 2 arguments: the string to be parsed and the radix. So, the above code is actually like calling: 

```javascript
// parse '1' with radix 0 
parseInt('1', 0, ['1','2','3']); // 1
// parse '2' with radix 1
parseInt('2', 1, ['1','2','3']); // NaN
// parse '3' with radix 2
parseInt('3', 2, ['1','2','3']); // NaN

// Gentle reminder, JavaScript just drops the extra parameter you pass in
```

As a post-mortem, what we should have done is to be explicit:

```javascript
['1','2','3'].map(x => parseInt(x, 10));
```

Goodbye point-free style.. :(


## 3. Using `Date` constructor without specifying time

There were many scenarios where I only needed a date instead of a datetime. I would conveniently/ ignorantly use the JavaScript's `Date` constructor passing in `String` in `YYYY-MM-dd` format. E.g. 

```javascript
new Date("2020-12-27")
// Sun Dec 27 2020 08:00:00 GMT+0800 (Singapore Standard Time)
```

Notice the time is set at 8:00 am instead of midnight. Seems okay if you're in Singapore. 

However, in today's globalized economy, it's rarely the case that your system only needs to support one timezone and the above code is actually creating a `Date` object with the time set to midnight at **UTC** timezone. So, if you run that very same code in the US, where it's behind the UTC, it will give you **yesterday's** date (i.e. 26 Dec 2020) in the local time:

```javascript
new Date("2020-12-27")
// Sat Dec 26 2020 16:00:00 GMT-0800 (Pacific Standard Time) {}

new Date("2020-12-27").toDateString()
// "Sat Dec 26 2020"
```

Whoops! This is definitely **not** what you intended! 

What should you do if you really need to get the date in the local time, though? Turns out, you just have to be explicit and supply the time (in ISO 8601 format) as well:

```javascript
new Date("2020-12-27T00:00:00")
// Sun Dec 27 2020 00:00:00 GMT-0800 (Pacific Standard Time)
```

Or, a better suggestion would perhaps be to use battle-tested libraries like [date-fns](https://date-fns.org/). 

## 4. Not Managing Node Proceses

This one is **not** strictly JavaScript; instead, it's more related to Node.js.

Unlike Tomcat with strong isolation of request threading & error handling, node process is very brittle and perhaps intentionally so. Any `uncaughtException` will cause the process to exit. 

Hence, it's almost never a good idea just to deploy barebone `node` to serve live traffic. All you need is one bad request and unhandled path to bring down your service. 

Instead, you should always use a process manager (e.g. [pm2](https://pm2.keymetrics.io/)) to automatically restarts your node server in case of crashes. And yes, crashing and restarting is counterintuitively the common pattern. This is because node server startup tends to be much faster than heavy weight containers like Tomcat, so the perceived downtime is minimal. 

## Epilogue
That's it for now! 

**PS**: If you're looking for a quick read on JavaScript fundamental and gotchas as you transition from being a backend to a full-stack developer, I'd strongly recommend purchasing my book: [The Minimum JavaScript You Should Know When You Code React & Redux](https://www.amazon.com/dp/B07G5YCRV2) on Amazon. 
