---
title: Testing Kafka Streams
date: 09-Mar-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: ["kafka", "streams", "topology", "testing"]
category: API Testing
categories: [API Testing, Kafka, Async, BDD]
excerpt: >-
  Learn how to validate stream processing topologies, joins, map operations, and windowed aggregations without a live broker.
readTime: 5 min read
---

# Blog #9: Testing Kafka Streams

Kafka Streams application transforms input topic streams to output topics.

## 1. Stream Topology


![diagram_1](https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/testing-kafka-streams-aggregations/images/diagram_1.png)


## 2. TopologyTestDriver

Testing stream processing with a live broker is slow. Confluent provides the `TopologyTestDriver` for lightning-fast local testing.

```java
Topology topology = builder.build();
Properties config = new Properties();
config.put(StreamsConfig.APPLICATION_ID_CONFIG, "test-stream");
config.put(StreamsConfig.BOOTSTRAP_SERVERS_CONFIG, "dummy:1234");
TopologyTestDriver testDriver = new TopologyTestDriver(topology, config);
```

## 3. Verifying Output

```java
TestInputTopic<String, String> inputTopic = testDriver.createInputTopic("input-topic", new StringSerializer(), new StringSerializer());
TestOutputTopic<String, String> outputTopic = testDriver.createOutputTopic("output-topic", new StringDeserializer(), new StringDeserializer());
inputTopic.pipeInput("key", "uppercase-me");
assertEquals("UPPERCASE-ME", outputTopic.readValue());
```
