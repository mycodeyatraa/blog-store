---
id: "post-737"
title: "Kafka Validation in Playwright Java"
slug: "kafka-validation-in-playwright-java"
date: "20-Aug-2026"
author: "pankaj-kumar"
series: "playwright-java-enterprise-data"
seriesOrder: 6
topic: "6. Event-Driven Kafka Stream Validation"
tags: ["Playwright", "Java", "Kafka", "Event Driven", "Messaging"]
category: "Automation Testing"
categories: ["Automation Testing", "Playwright Java", "Kafka"]
excerpt: "Verify event-driven message publishing on Apache Kafka topics triggered by Playwright Java browser user flows."
readTime: "8 min read"
---

# Kafka Validation in Playwright Java

Event-driven microservices publish events to Apache Kafka streams upon frontend user actions. Testing requires consuming Kafka messages to verify payload fidelity.

---

## 1. Architectural Overview

When Playwright Java triggers a UI workflow (e.g., placing an order), an event is produced onto a Kafka topic. The test runner uses `KafkaConsumer` to consume and assert message properties.

```
+------------------------------+       +------------------------------+
| Playwright Java Test         | ----> | Web Application UI           |
+------------------------------+       +------------------------------+
         |                                            |
         | Consume & Assert Messages                  | Produce Event
         v                                            v
+---------------------------------------------------------------------+
| Apache Kafka Cluster / Broker (Topic: order-events)                 |
+---------------------------------------------------------------------+
```

---

## 2. Kafka Consumer Helper

```java
// src/main/java/com/mycodeyatra/kafka/KafkaTestConsumer.java
package com.mycodeyatra.kafka;
 
import org.apache.kafka.clients.consumer.*;
import org.apache.kafka.common.serialization.StringDeserializer;
 
import java.time.Duration;
import java.util.*;
 
public class KafkaTestConsumer {
    private final KafkaConsumer<String, String> consumer;
 
    public KafkaTestConsumer(String bootstrapServers, String groupId, String topic) {
        Properties props = new Properties();
        props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapServers);
        props.put(ConsumerConfig.GROUP_ID_CONFIG, groupId);
        props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
        props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
        props.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "earliest");
 
        this.consumer = new KafkaConsumer<>(props);
        this.consumer.subscribe(Collections.singletonList(topic));
    }
 
    public String waitForMessageContaining(String keyword, int timeoutSeconds) {
        long end = System.currentTimeMillis() + (timeoutSeconds * 1000L);
        while (System.currentTimeMillis() < end) {
            ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(500));
            for (ConsumerRecord<String, String> record : records) {
                if (record.value().contains(keyword)) {
                    return record.value();
                }
            }
        }
        return null;
    }
 
    public void close() {
        consumer.close();
    }
}
```

---

## 3. Playwright Kafka Integration Test

```java
// src/test/java/com/mycodeyatra/tests/KafkaEventValidationTest.java
package com.mycodeyatra.tests;
 
import com.microsoft.playwright.*;
import com.mycodeyatra.kafka.KafkaTestConsumer;
import org.junit.jupiter.api.*;
 
import static org.junit.jupiter.api.Assertions.*;
 
public class KafkaEventValidationTest {
    private static Playwright playwright;
    private static Browser browser;
    private Page page;
    private KafkaTestConsumer kafkaConsumer;
 
    @BeforeAll
    static void setup() {
        playwright = Playwright.create();
        browser = playwright.chromium().launch(new BrowserType.LaunchOptions().setHeadless(true));
    }
 
    @BeforeEach
    void init() {
        page = browser.newPage();
        kafkaConsumer = new KafkaTestConsumer("localhost:9092", "test-group-" + System.currentTimeMillis(), "order-events");
    }
 
    @Test
    @DisplayName("Verify Order Checkout Publishes Kafka Event")
    void testKafkaEventOnCheckout() {
        String transactionId = "TXN-" + System.currentTimeMillis();
 
        // 1. Perform checkout UI action
        page.navigate("https://mycodeyatra.com/pay");
        page.fill("#txn-id", transactionId);
        page.click("#submit-pay-btn");
 
        // 2. Assert UI state
        assertTrue(page.isVisible("#payment-success"));
 
        // 3. Poll Kafka topic for matching payload event
        String eventPayload = kafkaConsumer.waitForMessageContaining(transactionId, 10);
        assertNotNull(eventPayload, "Kafka event containing transactionId must be published within timeout");
        assertTrue(eventPayload.contains("PAYMENT_COMPLETED"));
    }
 
    @AfterEach
    void cleanup() {
        if (kafkaConsumer != null) kafkaConsumer.close();
    }
 
    @AfterAll
    static void tearDown() {
        if (browser != null) browser.close();
        if (playwright != null) playwright.close();
    }
}
```

---

## 4. Enterprise Best Practices & Comparison Summary

- **Unique Consumer Group IDs**: Generate unique group IDs per test run to prevent offset collisions during parallel runs.
- **Poll Timeouts**: Implement bounded polling loops to gracefully fail tests if event streams stall.
