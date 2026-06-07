---
title: Mocking Kafka
date: 13-Mar-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: ["kafka", "mock", "unit-testing"]
category: API Testing
categories: [API Testing, Kafka, Async, BDD]
excerpt: >-
  Speed up unit testing! Learn how to use MockProducer and MockConsumer from kafka-clients for super-fast offline validation.
readTime: 5 min read
---

# Blog #13: Mocking Kafka

Sometimes you don't need a running docker broker. For unit tests, `MockProducer` and `MockConsumer` behave exactly like live clients but run entirely in memory.

## 1. MockProducer Example
```java
MockProducer<String, String> mockProducer = new MockProducer<>(true, new StringSerializer(), new StringSerializer());
mockProducer.send(new ProducerRecord<>("my-topic", "key", "value"));

// Verify immediately in-memory
List<ProducerRecord<String, String>> history = mockProducer.history();
assertEquals(1, history.size());
assertEquals("value", history.get(0).value());
```
This runs in a fraction of a millisecond!
