---
title: CI/CD Integration & Mocking with Karate
date: 16-Mar-2025
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: ["karate", "bdd", "ci-cd", "mocking", "github-actions"]
category: API Testing
categories: ["API Testing", "BDD", "Java", "Automation", "DevOps"]
excerpt: >-
  Master enterprise testing! Learn how to build native backend Mock Servers using Gherkin, and completely automate your test suite in a GitHub Actions CI/CD Pipeline.
readTime: 5 min read
---

# Blog #10: CI/CD Integration & Mocking with Karate

Welcome to the 10th and final part of the **Karate API Testing Series**!

To close out this masterclass, we will cover two advanced topics that elevate your framework to enterprise-grade: **Building Native Mock Servers** and **Integrating with GitHub Actions CI/CD**.

## 1. Native API Mocking in Karate
Sometimes, you need to test your service but the downstream dependencies (like a payment gateway) are down or too slow. Karate allows you to spin up a mock server using the exact same Gherkin syntax you use for testing!

### The Mock Server File
Create a `mock.feature` file. We define the routes using `pathMatches` and `methodIs`:

```gherkin
# src/test/java/com/mycodeyatra/karate/mock/mock.feature
Feature: Standalone Karate Mock Server

  Background:
    * configure cors = true

  Scenario: pathMatches('/greeting') && methodIs('get')
    * def response = { message: 'Hello from Karate Mock Server!' }
    * def responseStatus = 200

  Scenario: pathMatches('/users') && methodIs('post')
    * def response = { id: '#(java.util.UUID.randomUUID().toString())', status: 'created' }
    * def responseStatus = 201
```

### Programmatic Mock Runner
We can start this server dynamically in our JUnit suite on an ephemeral open port before executing our tests against it!

```java
// src/test/java/com/mycodeyatra/karate/mock/MockRunner.java
package com.mycodeyatra.karate.mock;

import com.intuit.karate.Results;
import com.intuit.karate.Runner;
import com.intuit.karate.core.MockServer;
import static org.junit.jupiter.api.Assertions.assertEquals;
import org.junit.jupiter.api.Test;

class MockRunner {
    @Test
    void testMock() {
        // Spin up the mock server on an open random port!
        MockServer server = MockServer.feature("classpath:.../mock.feature").http(0).build();
        int port = server.getPort();

        // Pass the port into our test feature
        Results results = Runner.path("classpath:.../test-mock.feature")
                .systemProperty("mockPort", port + "")
                .parallel(1);

        assertEquals(0, results.getFailCount());
        server.stop();
    }
}
```

## 2. CI/CD Integration with GitHub Actions
Running tests locally is great, but running them automatically on every Pull Request guarantees your codebase stays healthy.

Create `.github/workflows/karate-ci.yml`:

```yaml
name: Karate API Tests CI

on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    - name: Set up JDK 17
      uses: actions/setup-java@v3
      with:
        java-version: '17'
        distribution: 'temurin'
    - name: Run Karate Tests
      run: mvn clean test -Dtest=ParallelRunner
    - name: Upload HTML Reports
      uses: actions/upload-artifact@v3
      if: always()
      with:
        name: Karate-Test-Reports
        path: target/cucumber-html-reports/
```

With this, every code commit automatically spins up a clean Ubuntu runner, downloads Java 17, blasts through your tests in parallel, and zips up the beautiful HTML reports as an artifact you can download!

## 🔗 The Complete Masterclass Repository

You can access the entire source code, feature files, and configuration we built throughout this 10-part series at the official GitHub Repository:

👉 [**MyCodeYatra Karate Framework Repository**](https://github.com/MYCodeYatra/mcyt-api-karate)

Thank you so much for following along with this **10-Part Karate API Series** at MyCodeYatra. Happy Automating!
