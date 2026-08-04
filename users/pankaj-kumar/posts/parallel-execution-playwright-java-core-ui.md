---
title: Parallel Execution - Playwright Java Core UI
date: 21-Jan-2026
lastUpdated: 21-Jan-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [playwright, java, junit5, automation, ui-automation, mycodeyatra]
category: Playwright Java Core UI
categories: [Playwright Java Core UI, Playwright Java, Test Automation]
excerpt: >-
  Configure JUnit 5 parallel thread execution with isolated BrowserContexts for 10x faster regression suite runs.
readTime: 9 min read
---

# Parallel Execution - Playwright Java Core UI

Executing regression test suites sequentially is the biggest bottleneck in modern CI/CD pipelines. Playwright Java's lightweight `BrowserContext` architecture allows running dozens of tests concurrently without memory exhaustion.

---

## 1. JUnit 5 Parallel Configuration

Enable parallel test execution in JUnit 5 by creating `src/test/resources/junit-platform.properties`:

```properties
# Enable parallel test execution
junit.jupiter.execution.parallel.enabled=true
 
# Execute tests concurrently across classes and methods
junit.jupiter.execution.parallel.mode.default=concurrent
junit.jupiter.execution.parallel.mode.classes.default=concurrent
 
# Set thread pool strategy to dynamic CPU core multiplier
junit.jupiter.execution.parallel.config.strategy=dynamic
junit.jupiter.execution.parallel.config.dynamic.factor=1.0
```

---

## 2. Thread-Safe Base Test Architecture

```java
package com.mycodeyatra.tests;
 
import com.microsoft.playwright.*;
import org.junit.jupiter.api.*;
 
public abstract class ParallelBaseTest {
    private static Playwright playwright;
    private static Browser browser;
 
    // ThreadLocal ensures each test thread gets its own isolated context and page
    private final ThreadLocal<BrowserContext> threadContext = new ThreadLocal<>();
    private final ThreadLocal<Page> threadPage = new ThreadLocal<>();
 
    @BeforeAll
    static void startBrowser() {
        playwright = Playwright.create();
        browser = playwright.chromium().launch(new BrowserType.LaunchOptions().setHeadless(true));
    }
 
    @BeforeEach
    void initThreadContext() {
        BrowserContext context = browser.newContext();
        threadContext.set(context);
        threadPage.set(context.newPage());
    }
 
    protected Page getPage() {
        return threadPage.get();
    }
 
    @AfterEach
    void closeThreadContext() {
        threadPage.get().close();
        threadContext.get().close();
        threadPage.remove();
        threadContext.remove();
    }
 
    @AfterAll
    static void stopBrowser() {
        browser.close();
        playwright.close();
    }
}
```

---

## 3. Summary

Using `ThreadLocal<BrowserContext>` guarantees zero cross-test state leakage during high-speed parallel test runs.

