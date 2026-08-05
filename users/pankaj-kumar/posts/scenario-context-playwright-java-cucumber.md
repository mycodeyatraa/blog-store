---
title: "Sharing Scenario Context in Playwright Java Cucumber"
date: "15-Aug-2026"
lastUpdated: "15-Aug-2026"
author: "pankaj-kumar"
authorName: "Pankaj Kumar"
authorRole: "Automation Architect"
authorAvatar: "https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG"
authorBio: "Automation Architect"
authorGithub: "https://github.com/pankajhyd"
authorLinkedin: "https://www.linkedin.com/in/pankaj-kumar-94a2b227/"
tags: ["Playwright", "Java", "Cucumber", "PicoContainer", "Context"]
category: "Automation Testing"
categories: ["Automation Testing", "Playwright Java", "Scenario Context"]
excerpt: "Share dynamic state between step definitions using Dependency Injection (PicoContainer) in Playwright Java Cucumber automation frameworks."
readTime: "7 min read"
---

# Sharing Scenario Context in Playwright Java Cucumber

In production BDD frameworks, steps across different classes need to share runtime data (such as order IDs, generated tokens, or user session state) without using dangerous static variables.

---

## 1. Maven Dependency for PicoContainer

```xml
<dependency>
    <groupId>io.cucumber</groupId>
    <artifactId>cucumber-picocontainer</artifactId>
    <version>7.18.0</version>
    <scope>test</scope>
</dependency>
```

---

## 2. Scenario Context Container Class

```java
// src/test/java/com/mycodeyatra/context/TestContext.java
package com.mycodeyatra.context;
 
import java.util.HashMap;
import java.util.Map;
 
public class TestContext {
    private final Map<String, Object> sessionData = new HashMap<>();
 
    public void set(String key, Object value) {
        sessionData.put(key, value);
    }
 
    @SuppressWarnings("unchecked")
    public <T> T get(String key) {
        return (T) sessionData.get(key);
    }
 
    public boolean containsKey(String key) {
        return sessionData.containsKey(key);
    }
}
```

---

## 3. Injecting Context into Step Definition Classes

```java
// Step Class 1: Order Generation
package com.mycodeyatra.steps;
 
import com.mycodeyatra.context.TestContext;
import io.cucumber.java.en.When;
 
public class OrderSteps {
    private final TestContext context;
 
    public OrderSteps(TestContext context) {
        this.context = context;
    }
 
    @When("the user places an order for item {string}")
    public void placeOrder(String item) {
        String generatedOrderId = "ORD-" + System.currentTimeMillis();
        System.out.println("Generated Order ID: " + generatedOrderId);
        context.set("ORDER_ID", generatedOrderId);
    }
}
```

```java
// Step Class 2: Payment Verification
package com.mycodeyatra.steps;
 
import com.mycodeyatra.context.TestContext;
import io.cucumber.java.en.Then;
import static org.junit.jupiter.api.Assertions.assertNotNull;
 
public class PaymentSteps {
    private final TestContext context;
 
    public PaymentSteps(TestContext context) {
        this.context = context;
    }
 
    @Then("the payment invoice should contain the generated order ID")
    public void verifyInvoice() {
        String orderId = context.get("ORDER_ID");
        assertNotNull(orderId, "Order ID was not set in scenario context!");
        System.out.println("Verifying invoice for Order: " + orderId);
    }
}
```

---

## 4. Context Isolation Benefits

- **Thread Safety**: PicoContainer instantiates a new `TestContext` instance per scenario execution, ensuring parallel test threads never contaminate each other.
- **Zero Static State**: Eliminates fragile static variables that cause flaky test execution in multi-threaded runs.
