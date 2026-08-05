---
id: "post-746"
title: "Dashboard Metrics with Playwright Java"
slug: "dashboard-metrics-with-playwright-java"
date: "28-Aug-2026"
author: "pankaj-kumar"
series: "playwright-java-reporting-obs"
seriesOrder: 4
topic: "4. Executive Dashboard Metrics"
tags: ["Playwright", "Java", "Metrics", "Dashboard", "Grafana"]
category: "Automation Testing"
categories: ["Automation Testing", "Playwright Java", "Metrics"]
excerpt: "Capture metrics such as pass rates, execution duration, and flaky counts, converting execution data into Grafana / Kibana dashboard format."
readTime: "8 min read"
---

# Dashboard Metrics with Playwright Java

Executive dashboards require quantitative automation metrics: pass/fail rates, mean duration per scenario, and flakiness velocity.

---

## 1. Architectural Overview

Metrics collected during Playwright test runs are structured into JSON metric payloads and posted directly to Prometheus Pushgateway or Grafana HTTP endpoints.

```
+---------------------------------------+
| Playwright Java Suite                 |
| (Captures Execution Time & Status)    |
+---------------------------------------+
                    |
                    v HTTP POST Metrics Payload
+---------------------------------------+
| Prometheus Pushgateway / InfluxDB     |
+---------------------------------------+
                    |
                    v Query & Visualize
+---------------------------------------+
| Grafana Executive Dashboard           |
+---------------------------------------+
```

---

## 2. Dashboard Metrics Collector Utility

```java
// src/main/java/com/mycodeyatra/metrics/MetricsPublisher.java
package com.mycodeyatra.metrics;
 
import com.fasterxml.jackson.databind.ObjectMapper;
import java.net.URI;
import java.net.http.HttpClient;
import java.net.http.HttpRequest;
import java.net.http.HttpResponse;
import java.util.HashMap;
import java.util.Map;
 
public class MetricsPublisher {
    private static final HttpClient client = HttpClient.newHttpClient();
    private static final ObjectMapper mapper = new ObjectMapper();
 
    public static void publishMetric(String testName, long durationMs, String status) {
        try {
            Map<String, Object> payload = new HashMap<>();
            payload.put("test_name", testName);
            payload.put("duration_ms", durationMs);
            payload.put("status", status);
            payload.put("timestamp", System.currentTimeMillis());
 
            String json = mapper.writeValueAsString(payload);
 
            HttpRequest req = HttpRequest.newBuilder()
                    .uri(URI.create("http://localhost:9091/metrics/job/playwright_tests"))
                    .header("Content-Type", "application/json")
                    .POST(HttpRequest.BodyPublishers.ofString(json))
                    .build();
 
            client.sendAsync(req, HttpResponse.BodyHandlers.discarding());
        } catch (Exception e) {
            System.err.println("Metric publish warning: " + e.getMessage());
        }
    }
}
```

---

## 3. Playwright Metrics Integration Test

```java
// src/test/java/com/mycodeyatra/tests/MetricsValidationTest.java
package com.mycodeyatra.tests;
 
import com.microsoft.playwright.*;
import com.mycodeyatra.metrics.MetricsPublisher;
import org.junit.jupiter.api.*;
 
import static org.junit.jupiter.api.Assertions.*;
 
public class MetricsValidationTest {
    private static Playwright playwright;
    private static Browser browser;
    private Page page;
 
    @BeforeAll
    static void setup() {
        playwright = Playwright.create();
        browser = playwright.chromium().launch(new BrowserType.LaunchOptions().setHeadless(true));
    }
 
    @BeforeEach
    void init() {
        page = browser.newPage();
    }
 
    @Test
    @DisplayName("Measure Navigation Duration and Publish Metric")
    void testNavigationMetrics() {
        long start = System.currentTimeMillis();
        String status = "PASSED";
 
        try {
            page.navigate("https://mycodeyatra.com/docs");
            assertTrue(page.isVisible("h1"));
        } catch (Throwable t) {
            status = "FAILED";
            throw t;
        } finally {
            long duration = System.currentTimeMillis() - start;
            MetricsPublisher.publishMetric("testNavigationMetrics", duration, status);
        }
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

- **Non-Blocking Publishing**: Send metrics asynchronously (`sendAsync`) so network hiccups do not slow down test execution.
- **Standardized Tags**: Tag metrics with branch name, commit hash, and browser name for granular Grafana filtering.
