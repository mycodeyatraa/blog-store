---
title: Dead Letter Queue (DLQ) Testing
date: 10-Mar-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: ["kafka", "dlq", "error-handling"]
category: API Testing
categories: [API Testing, Kafka, Async, BDD]
excerpt: >-
  Ensure reliability! Validate error-handling mechanisms and ensure corrupt payloads are routed to retry and DLQ topics.
readTime: 5 min read
---

# Blog #10: Dead Letter Queue (DLQ) Testing

What happens when your consumer receives a corrupt or invalid JSON payload? It shouldn't crash the server. It should route it to a **DLQ**.

## 1. Testing DLQ Routing

We can write an automation test to explicitly send bad payloads and verify that:
1. The consumer safely logs the parsing exception.
2. The consumer publishes the failed payload to `user-events-dlq`.

## 2. Asserting on the DLQ Topic
```java
// Publish bad payload
producer.send(new ProducerRecord<>("user-events", "user-1", "INVALID_JSON_HERE"));

// Verify it ends up in DLQ using Awaitility
Awaitility.await().atMost(5, TimeUnit.SECONDS).untilAsserted(() -> {
    ConsumerRecord<String, String> dlqRecord = dlqConsumer.poll(Duration.ofMillis(500)).iterator().next();
    assertEquals("INVALID_JSON_HERE", dlqRecord.value());
});
```
