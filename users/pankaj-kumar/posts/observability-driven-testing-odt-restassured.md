---
title: Observability-Driven Testing (ODT) in REST-Assured
date: 23-Feb-2026
lastUpdated: 23-Feb-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: ["restassured", "java", "observability", "odt", "opentelemetry", "tracing"]
category: REST-Assured
categories: ["REST-Assured", "API Testing", "Java"]
excerpt: >-
  Shift from basic HTTP status checks to trace-based assertions! Implement Observability-Driven Testing with OpenTelemetry and REST-Assured.
readTime: 8 min read
---

# Observability-Driven Testing (ODT) in REST-Assured

In complex distributed microservice architectures, an API endpoint might return an HTTP `200 OK` status even if a background database write timed out or an asynchronous Kafka event was dropped.

**Observability-Driven Testing (ODT)** shifts test validation beyond HTTP headers. By injecting OpenTelemetry **Trace IDs** (`traceparent`) into REST-Assured requests, test automation suites can query backend telemetry stores (Jaeger, Zipkin, Datadog) to verify internal microservice span operations.

---

## 1. Trace Injection Architecture

```
 +------------------------+   HTTP + traceparent   +--------------------+
 | REST-Assured Test Run  | ---------------------> | Gateway Service    |
 +------------------------+                        +--------------------+
             |                                                |
      Assert Trace Query                               Spans Propagated
             |                                                v
             v                                     +--------------------+
 +------------------------+                        | Backend Microservice|
 | OpenTelemetry / Jaeger | <---(Spans Ingested)---| (DB, Kafka, Cache) |
 +------------------------+                        +--------------------+
```

---

## 2. Implementing ODT Header Injection

Inject standard W3C `traceparent` headers into REST-Assured calls:

**utils/OdtTraceFilter.java**

```java
package com.mycodeyatra.utils;
 
import io.restassured.filter.Filter;
import io.restassured.filter.FilterContext;
import io.restassured.response.Response;
import io.restassured.specification.FilterableRequestSpecification;
import io.restassured.specification.FilterableResponseSpecification;
 
import java.util.UUID;
 
public class OdtTraceFilter implements Filter {
 
    @Override
    public Response filter(FilterableRequestSpecification requestSpec,
                           FilterableResponseSpecification responseSpec,
                           FilterContext ctx) {
        
        String traceId = UUID.randomUUID().toString().replace("-", "");
        String spanId = UUID.randomUUID().toString().substring(0, 16).replace("-", "");
        String traceparent = String.format("00-%s-%s-01", traceId, spanId);
 
        requestSpec.header("traceparent", traceparent);
        requestSpec.header("X-Correlation-ID", traceId);
 
        System.out.println("[ODT Filter] Injected Trace ID: " + traceId);
        
        return ctx.next(requestSpec, responseSpec);
    }
}
```

---

## 3. Executing Observability-Driven API Tests

**src/test/java/com/mycodeyatra/OdtApiTest.java**

```java
package com.mycodeyatra;
 
import com.mycodeyatra.utils.OdtTraceFilter;
import io.restassured.RestAssured;
import io.restassured.http.ContentType;
import org.junit.jupiter.api.Test;
 
import static io.restassured.RestAssured.given;
import static org.hamcrest.Matchers.equalTo;
 
public class OdtApiTest {
 
    @Test
    public void testApiWithTraceCorrelation() {
        given()
            .baseUri("https://mycodeyatra.com/api")
            .filter(new OdtTraceFilter())
            .contentType(ContentType.JSON)
        .when()
            .get("/orders/1001")
        .then()
            .statusCode(200)
            .body("status", equalTo("PROCESSED"));
    }
}
```
