---
title: Writing your first Kafka Consumer Test
date: 05-Mar-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: ["kafka", "consumer", "java", "testing"]
category: API Testing
categories: [API Testing, Kafka, Async, BDD]
excerpt: >-
  Learn how to set up a subscription poll loop, fetch messages, and assert their key and value payloads in automated tests.
readTime: 5 min read
---

# Blog #5: Writing your first Kafka Consumer Test

Consuming messages is slightly different due to the polling loop.

## 1. Consumer Loop Flow

Consumers poll Kafka brokers inside a loop to pull messages in batches:


![diagram_1](https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/writing-first-kafka-consumer-test/images/diagram_1.png)


## 2. Consumer Configuration

Here is the setup configuration required to consume messages in Java:

```java
Properties props = new Properties();
props.put("bootstrap.servers", "localhost:29092");
props.put("group.id", "test-verifier-group");
props.put("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
props.put("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
props.put("auto.offset.reset", "earliest");
Consumer<String, String> consumer = new KafkaConsumer<>(props);
consumer.subscribe(Collections.singletonList("mycodeyatra-events"));
```

## 3. Poll and Assert
We poll for records and assert on the payload:

```java
ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(5000));
boolean found = false;
for (ConsumerRecord<String, String> record : records) {
    if (record.value().equals("Hello Kafka!")) {
        found = true;
        break;
    }
}
assertTrue(found, "The expected message was not consumed!");
```
