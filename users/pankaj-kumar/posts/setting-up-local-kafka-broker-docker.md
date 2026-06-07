---
title: Setting up a Local Kafka Broker
date: 02-Mar-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: ["kafka", "docker", "setup", "infrastructure"]
category: API Testing
categories: [API Testing, Kafka, Async, BDD]
excerpt: >-
  Learn how to easily spin up a multi-container local testing environment containing Apache Kafka and Zookeeper using Docker Compose.
readTime: 5 min read
---

# Blog #2: Setting up a Local Kafka Broker

To write automated tests against Apache Kafka, we need a reliable, isolated broker running locally. The industry-standard way to accomplish this is using **Docker Compose**.

## 1. Network Architecture

Understanding listener configuration is crucial. The host machine (running your test suite) connects via a different port than other containers running in the same Docker network:


![diagram_1](https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/setting-up-local-kafka-broker-docker/images/diagram_1.png)


## 2. The docker-compose.yml Setup

Here is a standard, robust configuration spinning up Zookeeper and Kafka:

```yaml
version: '3.8'
services:
  zookeeper:
    image: confluentinc/cp-zookeeper:7.5.0
    container_name: myct-zookeeper
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181
      ZOOKEEPER_TICK_TIME: 2000
    ports:
      - "2181:2181"

  kafka:
    image: confluentinc/cp-kafka:7.5.0
    container_name: myct-kafka
    ports:
      - "9092:9092"
      - "29092:29092"
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka:9092,PLAINTEXT_HOST://localhost:29092
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
```

## 3. Understanding Advertised Listeners
* **PLAINTEXT://kafka:9092**: Used for internal communication inside the Docker network (e.g., from other containerized services like our Node mock server).
* **PLAINTEXT_HOST://localhost:29092**: Exposed to your host system so your local IDE and testing frameworks (Maven, Node) can reach Kafka directly.
