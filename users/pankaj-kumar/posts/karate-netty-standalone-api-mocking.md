---
title: Karate Netty: Mocking APIs Easily with Karate Standalone
date: 24-Feb-2026
lastUpdated: 24-Feb-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: ["karate", "netty", "mocking", "api-testing", "standalone"]
category: API Karate
categories: ["API Karate", "API Testing", "Mocking"]
excerpt: >-
  Build high-performance mock servers without Java server code! Harness Karate Netty to mock HTTP endpoints dynamically.
readTime: 8 min read
---

# Karate Netty: Mocking APIs Easily with Karate Standalone

In modern microservice development, backend APIs are often being built concurrently with consumer applications. **Karate Netty** allows test engineers to spin up lightweight, high-performance HTTP mock servers using pure Gherkin feature syntax—without writing any Java web controller code.

---

## 1. Core Architecture of Karate Netty Server

Karate Netty runs a standalone HTTP server engine powered by Netty that listens for requests, matches Gherkin `Scenario` conditions, and returns stubbed JSON or XML responses dynamically.

```
 +--------------------+      HTTP Request       +--------------------------+
 | Consumer Client /  | ----------------------> | Karate Netty Server      |
 | Automated Test     | <---------------------- | (mock.feature on :8080)  |
 +--------------------+      Stubbed Response   +--------------------------+
```

---

## 2. Defining a Mock Server Feature File

Below is a complete Karate Netty mock server definition:

**mocks/user-service-mock.feature**

```gherkin
Feature: Standalone User Service Mock Server
 
  Background:
    * def users = { '101': { id: 101, name: 'Pankaj Kumar', role: 'Architect' } }
 
  Scenario: pathMatches('/api/users/{id}') && methodIs('get')
    * def id = pathParams.id
    * def user = users[id]
    * def response = user ? user : { error: 'User Not Found' }
    * def responseStatus = user ? 200 : 404
 
  Scenario: pathMatches('/api/users') && methodIs('post')
    * def newUser = request
    * def newId = '102'
    * newUser.id = newId
    * users[newId] = newUser
    * def response = newUser
    * def responseStatus = 201
```

---

## 3. Running and Testing Against the Mock Server

Execute the mock server programmatically or via CLI:

**src/test/java/com/mycodeyatra/MockServerRunner.java**

```java
package com.mycodeyatra;
 
import com.intuit.karate.core.MockServer;
import org.junit.jupiter.api.AfterAll;
import org.junit.jupiter.api.BeforeAll;
import org.junit.jupiter.api.Test;
 
import static com.intuit.karate.Runner.runFeature;
 
public class MockServerRunner {
 
    private static MockServer server;
 
    @BeforeAll
    public static void startMockServer() {
        // Spin up Karate Netty mock server on port 8085
        server = MockServer.feature("classpath:mocks/user-service-mock.feature")
                           .http(8085)
                           .build();
    }
 
    @AfterAll
    public static void stopMockServer() {
        server.stop();
    }
 
    @Test
    public void testMockApiConsumer() {
        runFeature("classpath:features/consume-mock.feature", null, true);
    }
}
```

---

## Key Benefits

1. **Zero Boilerplate**: No need to configure Spring Boot, WireMock, or Express.js servers.
2. **Deterministic State**: Easily update internal JS mock state per scenario iteration.
