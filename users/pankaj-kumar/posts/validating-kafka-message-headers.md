---
title: Validating Message Headers
date: 07-Mar-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: ["kafka", "headers", "metadata"]
category: API Testing
categories: [API Testing, Kafka, Async, BDD]
excerpt: >-
  Test your metadata. Learn how to verify correlation IDs, partition keys, and tracing metadata stored in Kafka headers.
readTime: 5 min read
---

# Blog #7: Validating Message Headers

Kafka messages aren't just key-value pairs; they also contain headers. Headers are crucial for tracing and routing.

## 1. Adding Headers in Producer

```java
ProducerRecord<String, String> record = new ProducerRecord<>("user-events", "user-1", "Payload");
record.headers().add("correlation-id", "xyz-789-abc".getBytes());
record.headers().add("source-app", "api-server".getBytes());
```

## 2. Validating Headers in Consumer Test
In your assertions, check that tracing properties exist:

```java
Header correlationIdHeader = record.headers().lastHeader("correlation-id");
assertNotNull(correlationIdHeader);
String correlationId = new String(correlationIdHeader.value());
assertEquals("xyz-789-abc", correlationId);
```
