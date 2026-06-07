---
title: Testcontainers for Kafka
date: 12-Mar-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: ["kafka", "testcontainers", "junit", "ci"]
category: API Testing
categories: [API Testing, Kafka, Async, BDD]
excerpt: >-
  Say goodbye to pre-configured brokers! Learn how to spin up ephemeral, lightweight Kafka containers on-the-fly inside JUnit.
readTime: 5 min read
---

# Blog #12: Testcontainers for Kafka

Hardcoded local brokers can fail in CI/CD pipelines if they are not running. **Testcontainers** solves this by spinning up a clean Docker container directly from JUnit.

## 1. JUnit Docker Orchestration


![diagram_1](https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/using-testcontainers-for-ephemeral-kafka/images/diagram_1.png)


## 2. Setting up Testcontainers Kafka

```java
@Container
public static KafkaContainer kafka = new KafkaContainer(DockerImageName.parse("confluentinc/cp-kafka:7.5.0"));
```

## 3. Fetching Dynamic Ports

Because Testcontainers maps random ports to prevent collisions, you inject the port dynamically at runtime:

```java
String bootstrapServers = kafka.getBootstrapServers();
props.put("bootstrap.servers", bootstrapServers);
```
This ensures your tests run reliably on any local or remote machine without manual environment setup.
