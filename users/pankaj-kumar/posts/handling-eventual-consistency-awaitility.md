---
title: Handling Eventual Consistency
date: 08-Mar-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: ["kafka", "testing", "awaitility", "async"]
category: API Testing
categories: [API Testing, Kafka, Async, BDD]
excerpt: >-
  Say goodbye to Thread.sleep()! Master wait strategies and async assertions using Awaitility for asynchronous testing.
readTime: 5 min read
---

# Blog #8: Handling Eventual Consistency

Asynchronous tests are prone to **flakiness** if you use arbitrary waits like `Thread.sleep()`.

## 1. Enter Awaitility
Awaitility allows you to express expectations of an asynchronous system in a clean, readable DSL.

```java
// Maven dependency
// <dependency>
//     <groupId>org.awaitility</groupId>
//     <artifactId>awaitility</artifactId>
//     <version>4.2.0</version>
// </dependency>
```

## 2. Using Async Assertions
Instead of a fixed sleep, we wait dynamically until the message is received:

```java
Awaitility.await()
    .atMost(5, TimeUnit.SECONDS)
    .pollInterval(100, TimeUnit.MILLISECONDS)
    .untilAsserted(() -> {
        List<String> messages = consumerHelper.getReceivedMessages();
        assertTrue(messages.contains("Hello Kafka!"));
    });
```
This speeds up successful tests and prevents random test failures!
