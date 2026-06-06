---
title: OpenTelemetry Integration with RestAssured
date: 17-Feb-2025
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [restassured, java, opentelemetry, tracing, odt]
category: API Testing
categories: [API Testing, Observability, Automation, Java]
excerpt: >-
  Learn how to inject W3C Trace Context headers into your RestAssured tests to initiate distributed OpenTelemetry traces across your backend microservices.
readTime: 6 min read
---

In our previous tutorial, we explored the theory behind Observability-Driven Testing (ODT). Now, it's time to get our hands dirty and implement it using the industry standard for telemetry: **OpenTelemetry (OTel)**.

OpenTelemetry provides a unified standard for collecting logs, metrics, and traces across distributed systems. By integrating OTel into our RestAssured tests, we can verify that our tests are properly participating in the system's distributed trace.

---

## 1. The Goal of Trace Injection

When RestAssured sends an HTTP Request to an API Gateway, we want to ensure that the trace initiated by the test propagates perfectly through the entire backend architecture.

To do this, we inject a unique `traceparent` header into our HTTP Request. This conforms to the **W3C Trace Context specification**.

## 2. Adding OpenTelemetry Dependencies

First, we need to add the OpenTelemetry Java SDK to our `pom.xml`.

```xml
<dependency>
    <groupId>io.opentelemetry</groupId>
    <artifactId>opentelemetry-api</artifactId>
    <version>1.36.0</version>
</dependency>
<dependency>
    <groupId>io.opentelemetry</groupId>
    <artifactId>opentelemetry-sdk</artifactId>
    <version>1.36.0</version>
</dependency>
```

## 3. Creating a Trace Injection Filter

RestAssured provides a powerful `Filter` interface that allows us to intercept and modify HTTP Requests before they are sent. We can create an `OpenTelemetryFilter` that automatically generates a W3C `traceparent` header and attaches it to every outgoing test request!

```java
package com.mycodeyatra.framework;
import io.opentelemetry.api.trace.Span;
import io.opentelemetry.api.trace.SpanContext;
import io.opentelemetry.api.trace.TraceFlags;
import io.opentelemetry.api.trace.TraceState;
import io.restassured.filter.Filter;
import io.restassured.filter.FilterContext;
import io.restassured.response.Response;
import io.restassured.specification.FilterableRequestSpecification;
import io.restassured.specification.FilterableResponseSpecification;
import java.util.UUID;
public class OpenTelemetryFilter implements Filter {
    @Override
    public Response filter(FilterableRequestSpecification requestSpec, 
                           FilterableResponseSpecification responseSpec, 
                           FilterContext ctx) {
        // 1. Generate a unique 32-character Trace ID for this specific test request
        String traceId = UUID.randomUUID().toString().replace("-", "") + 
                         UUID.randomUUID().toString().replace("-", "").substring(0, 16);
        // 2. Generate a 16-character Span ID
        String spanId = UUID.randomUUID().toString().replace("-", "").substring(0, 16);
        // 3. Format according to W3C Trace Context (00-traceid-spanid-01)
        String traceparent = String.format("00-%s-%s-01", traceId, spanId);
        // 4. Inject the header into the RestAssured Request
        requestSpec.header("traceparent", traceparent);
        System.out.println("Injected W3C Trace Context: " + traceparent);
        // Continue executing the request
        return ctx.next(requestSpec, responseSpec);
    }
}
```

## 4. Applying the Filter to the Test

Now, we simply attach this filter to our RestAssured specification!

```java
import org.testng.annotations.Test;
import static io.restassured.RestAssured.given;
public class OpenTelemetryTest {
    @Test
    public void testWithOtelTraceInjection() {
        given()
            .baseUri("http://localhost:8080")
            .filter(new OpenTelemetryFilter()) // Apply the Telemetry Filter!
        .when()
            .get("/api/users/1")
        .then()
            .statusCode(200);
    }
}
```

## 5. What Happens Next?

When this test runs, RestAssured fires the HTTP Request with a header like `traceparent: 00-4bf92f3577b34da6a3ce929d0e0e4736-00f067aa0ba902b7-01`.

The backend SpringBoot, NodeJS, or Go microservices detect this W3C header and automatically adopt the Trace ID! They will continue propagating it down to the database level. 

After the test completes, you can log into your Jaeger, Zipkin, or DataDog dashboard, search for that exact Trace ID, and see the full waterfall execution of your automated test!

In our final tutorial, we will take this a step further using **Tracetest** to actually *write assertions* against those backend database spans directly inside Java!
