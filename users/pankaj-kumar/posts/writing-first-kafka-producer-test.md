---
title: Writing your first Kafka Producer Test
date: 04-Mar-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: ["kafka", "producer", "java", "testing"]
category: API Testing
categories: [API Testing, Kafka, Async, BDD]
excerpt: >-
  Step-by-step guide to writing a Kafka producer client and verifying that messages are successfully published to topics.
readTime: 5 min read
---

# Blog #4: Writing your first Kafka Producer Test

Now let's write code to publish a message and verify it!

## 1. Java Kafka Producer Properties

To connect to our Dockerized Kafka broker, we define producer properties:

```java
Properties props = new Properties();
props.put("bootstrap.servers", "localhost:29092");
props.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
props.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");

Producer<String, String> producer = new KafkaProducer<>(props);
```

## 2. Sending and Validating
We send a record using `ProducerRecord` and verify the metadata:

```java
ProducerRecord<String, String> record = new ProducerRecord<>("mycodeyatra-events", "key-1", "Hello Kafka!");
RecordMetadata metadata = producer.send(record).get(); // Blocks for verification

System.out.println("Message written to partition: " + metadata.partition());
System.out.println("Offset: " + metadata.offset());
```
In our tests, we assert that the `offset` is greater than or equal to 0, confirming the message was stored.
