---
title: Schema Registry Validation
date: 06-Mar-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: ["kafka", "schema", "avro", "validation"]
category: API Testing
categories: [API Testing, Kafka, Async, BDD]
excerpt: >-
  Ensure message integrity! Learn how to test event formats (Avro, Protobuf, JSON) against a Confluent Schema Registry.
readTime: 5 min read
---

# Blog #6: Schema Registry Validation

In enterprise software, free-form JSON strings lead to broken consumer applications. We use schemas to enforce data structures.

## 1. Confluent Schema Registry

The schema registry acts as a shared dictionary of schemas. Producers register schemas, and consumers fetch them to deserialize payloads.


![diagram_1](https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/schema-registry-validation-avro-protobuf/images/diagram_1.png)


## 2. Testing Schema Compatibility

In your test suite, you configure Avro serializers to talk to the registry URL:

```java
Properties props = new Properties();
props.put("bootstrap.servers", "localhost:29092");
props.put("key.serializer", "io.confluent.kafka.serializers.KafkaAvroSerializer");
props.put("value.serializer", "io.confluent.kafka.serializers.KafkaAvroSerializer");
props.put("schema.registry.url", "http://localhost:8081");
```

By asserting that schema validation does not throw serialization errors, you guarantee compatibility before deployment.
