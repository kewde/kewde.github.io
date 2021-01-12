---
layout: post
date: 2020-12-15 09:00:00 UTC
title: Hacking Bitcoin Wallets Through Differences In JSON Parsers
description: How a subtle bug in a parser can leave your wallet empty.
---

# TL;DR
I didn't steal any bitcoin. I've reported it to the appropriate company and they were shocked at how a bug this simple could have left their wallets empty.
Using different libraries to parse formats like JSON is a bad idea and can expose you to a type of attack that relies on different interpretations of the content by each parser. The microservice pattern introduces the ability to use amalgation of programming languages, but that typically also means differences in parsers.

# Microservice Pattern
There's a lot of love for the microservice pattern and rightfully so. In essence, the microservice pattern tends to lead to greater seperation of privileges and access.
The code that decodes media objects such as videos and images runs in its own microservice, seperating it from the code that can access the database storing your VISA card credentials for example. 

But the advocacy for the microservice pattern isn't only coming from the security perspective. 
> The microservices architecture allowed Netflix to greatly speed up development and deployment of its platform and services. The company was able to build and test global services on a large scale without impacting the current system and they could quickly rollback if there were problems. The microservices architecture also allowed Netflix to create about 30+ independent engineering teams that could work on different release schedules which helped increase the agility and productivity of the development process. [0]


The main driver for organisations to adopt the microservice pattern is to enable faster development and deployments because each team operates more independently.
Every team can pick their own stack (language, libraries, etc..) to use in order to optimize their particular workload.

# Two steps forward, but also one step back
I consider it to be a better pattern than the monolith if applied correctly.
But the step back is that it may breath new life into an age-old security bug: parser differences.

Here's my advice for anyone out there building microservices:
> The JSON format leaves gaps for interpretation so make sure that all parsers across all microservices are using a single way to interpret the content.

If you don't quite catch what I'm trying to explain, here's an example:
```
{
    "method": "sendtoaddress",
    "method": "listtransactions",
    "params": [...]
}
```

Notice how the `method` key is presented twice in this JSON payload, but which method of the two will it eventually execute?
This the gap for interpretation that parsers silently deal with. This rather silent assumption can be a great attack vector.

Can we abuse the difference in interpretation by parser X used in microservice 1, which feeds data to parser Y in microservice 2?

# Stealing Bitcoins Because Of A Parser Bug

This is an actual bugreport that I've submitted to a business that operates cryptocurrency nodes that a customer can rent.
But one of their options allows exposing a limited interface of `bitcoind`, behind a custom firewall that blocks sensitive methods.
```
USER   ----> [ FIREWALL ]    ---->    [ bitcoind ]
```

The firewall microservice maintained a whitelist of methods (`getblock`, etc..) which was supposed to reject attempts to execute senstitive or malicious commands.
If it passed the filter, it would then pass the *COMPLETE* payload to the `bitcoind` daemon, which would then execute and return the results.

Here's an overview to two parsers at play:
* firewall: LIFO (OpenResty from **Lua**)
* bitcoind: FIFO (Boost (?) json parser from **C++**)


Let's for example attempt to get the balance of the bitcoind node:
```
{
    "method": "getbalance"
}
```

We are met with the following response:
```
403 Forbidden
```

At this point I wondered whether we could bypass it, and naturally the first idea that I could come up with was setting it twice.
```
{
    "method": "getblock",
    "method": "getbalance"
}
```
```
403 Forbidden
```
It failed, but luckily I attempted to switch them around, just to be sure.
This time we hit the jackpot!

```
{
    "method": "getbalance",
    "method": "getblock"
}
```
It returns the getbalance information instead of the error!
```
{
    "result": 5.35784803,
    "error": null,
    "id": null
}
```

The firewall is using a `LIFO` style parser, which returned `getblock` as the value of the key `method`. It accepted the request as valid and forwarded it to the `bitcoind` daemon which uses a `FIFO` style parser resulting in `getbalance` to be used as the value of the key `method`

Nice. 

# References:

* 0: https://smartbear.com/blog/develop/why-you-cant-talk-about-microservices-without-ment/


