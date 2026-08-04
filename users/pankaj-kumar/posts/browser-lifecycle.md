---
title: Browser Lifecycle - Playwright Java Foundations
date: 06-Jan-2026
lastUpdated: 06-Jan-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [playwright, java, junit5, automation, testing, mycodeyatra]
category: Playwright Java Foundations
categories: [Playwright Java Foundations, Playwright Java, Test Automation]
excerpt: >-
  Master Playwright, Browser, BrowserContext, and Page lifecycles to achieve 100% thread-safe parallel test isolation.
readTime: 9 min read
---

# Browser Lifecycle - Playwright Java Foundations

In modern automated testing, state pollution between tests (such as leftover cookies or session storage) causes non-deterministic flakiness. Playwright Java solves this with its lightweight `BrowserContext` lifecycle model.

---

## 1. The Power of BrowserContext Isolation

In legacy Selenium frameworks, achieving test isolation required launching a brand new browser process (`new ChromeDriver()`) for every test—a slow operation consuming hundreds of megabytes of RAM.

```
SELENIUM APPROACH (Slow & Heavy):
Test 1 -> New Chrome Process (RAM: 400MB, Time: 3s)
Test 2 -> New Chrome Process (RAM: 400MB, Time: 3s)
 
PLAYWRIGHT APPROACH (Fast & Lightweight):
Single Browser Process (RAM: 200MB)
 ├── Context 1 (RAM: 5MB, Time: 15ms) -> Test 1
 └── Context 2 (RAM: 5MB, Time: 15ms) -> Test 2
```

A `BrowserContext` operates like an incognito window. It has its own:
- Cookies and HTTP Headers
- LocalStorage and SessionStorage
- Cache and IndexedDB databases
- Permissions and Geolocation settings

---

## 2. Production Lifecycle Base Class (`src/test/java/com/mycodeyatra/tests/BaseTest.java`)

Here is the standard JUnit 5 lifecycle class used in enterprise frameworks:

```java
package com.mycodeyatra.tests;
 
import com.microsoft.playwright.*;
import org.junit.jupiter.api.*;
 
public abstract class BaseTest {
    protected static Playwright playwright;
    protected static Browser browser;
    protected BrowserContext context;
    protected Page page;
 
    @BeforeAll
    static void launchBrowser() {
        playwright = Playwright.create();
        browser = playwright.chromium().launch(new BrowserType.LaunchOptions().setHeadless(true));
    }
 
    @BeforeEach
    void createContext() {
        // Creates a fresh incognito context per test
        context = browser.newContext(new Browser.NewContextOptions()
            .setViewportSize(1920, 1080)
            .setIgnoreHTTPSErrors(true));
        page = context.newPage();
    }
 
    @AfterEach
    void closeContext() {
        context.close(); // Instantly clears cookies and local storage
    }
 
    @AfterAll
    static void closeBrowser() {
        browser.close();
        playwright.close();
    }
}
```

---

## 3. Key Takeaways

1. **Never Share Contexts Across Tests**: Always call `browser.newContext()` inside `@BeforeEach`.
2. **Explicit Teardown**: Always close contexts in `@AfterEach` to free up system memory immediately.

