---
id: "post-754"
title: "Log Aggregation (Logstash) with Playwright Java"
slug: "log-aggregation-logstash-with-playwright-java"
date: "05-Sep-2026"
author: "pankaj-kumar"
series: "playwright-java-reporting-obs"
seriesOrder: 12
topic: "12. Log Aggregation & Logstash Pipeline"
tags: ["Playwright", "Java", "Logstash", "ELK", "Log4j2"]
category: "Automation Testing"
categories: ["Automation Testing", "Playwright Java", "Logstash"]
excerpt: "Stream structured JSON logs from Playwright Java using Log4j2 directly into Logstash and the ELK stack."
readTime: "8 min read"
---

# Log Aggregation (Logstash) with Playwright Java

Centralizing automation logs in an ELK (Elasticsearch, Logstash, Kibana) stack allows enterprise teams to search across thousands of test runs instantly.

---

## 1. Architectural Overview

Using Log4j2 with `SocketAppender` or `JSONLayout`, Playwright Java logs are streamed formatted as JSON to a Logstash TCP input port.

```
+--------------------------------------+       +------------------------------------+
| Playwright Java Test Runner          | ----> | Logstash TCP Input (Port 5000)     |
| (Log4j2 SocketAppender JSON Format)  |       +------------------------------------+
+--------------------------------------+                          |
                                                                  v Indexes Logs
+-----------------------------------------------------------------------------------+
| Elasticsearch Engine & Kibana Log Visualization Dashboard                         |
+-----------------------------------------------------------------------------------+
```

---

## 2. Log4j2 JSON Socket Appender Configuration

```xml
<!-- src/main/resources/log4j2.xml snippet -->
<Configuration status="WARN">
    <Appenders>
        <Socket name="Logstash" host="localhost" port="5000">
            <JsonLayout compact="true" eventEol="true"/>
        </Socket>
        <Console name="Console" target="SYSTEM_OUT">
            <PatternLayout pattern="%d{HH:mm:ss.SSS} [%t] %-5level %logger{36} - %msg%n"/>
        </Console>
    </Appenders>
    <Loggers>
        <Root level="info">
            <AppenderRef ref="Logstash"/>
            <AppenderRef ref="Console"/>
        </Root>
    </Loggers>
</Configuration>
```

---

## 3. Playwright Logstash Streaming Test Suite

```java
// src/test/java/com/mycodeyatra/tests/LogstashAggregationTest.java
package com.mycodeyatra.tests;
 
import com.microsoft.playwright.*;
import org.apache.logging.log4j.LogManager;
import org.apache.logging.log4j.Logger;
import org.junit.jupiter.api.*;
 
import static org.junit.jupiter.api.Assertions.*;
 
public class LogstashAggregationTest {
    private static final Logger logger = LogManager.getLogger(LogstashAggregationTest.class);
    private static Playwright playwright;
    private static Browser browser;
    private Page page;
 
    @BeforeAll
    static void setup() {
        playwright = Playwright.create();
        browser = playwright.chromium().launch(new BrowserType.LaunchOptions().setHeadless(true));
        logger.info("Playwright Browser Instance Launched for Logstash Streaming");
    }
 
    @BeforeEach
    void init() {
        page = browser.newPage();
    }
 
    @Test
    @DisplayName("Stream Test Step Logs to Logstash")
    void testLogstashStreaming() {
        logger.info("Navigating to target application URL");
        page.navigate("https://mycodeyatra.com");
 
        logger.info("Asserting page title visibility");
        assertEquals("MyCodeYatra", page.title());
        logger.info("Test Assertion Passed Successfully");
    }
 
    @AfterAll
    static void tearDown() {
        if (browser != null) browser.close();
        if (playwright != null) playwright.close();
        logger.info("Playwright Browser Shutdown Completed");
    }
}
```

---

## 4. Enterprise Best Practices & Comparison Summary

- **Structured Fields**: Include MDC (Mapped Diagnostic Context) parameters like `testId`, `environment`, and `buildNumber` in every log entry.
- **Asynchronous Logging**: Enable Log4j2 AsyncLoggers to ensure log socket I/O never delays test execution.
