---
title: Assertions - Playwright Java Foundations
date: 08-Jan-2026
lastUpdated: 08-Jan-2026
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
  Master web-first assertions in Playwright Java using PlaywrightAssertions.assertThat() with automatic polling and retries.
readTime: 9 min read
---

# Assertions - Playwright Java Foundations

Traditional assertions in testing frameworks (such as `assertEquals` or `assertTrue` in JUnit) evaluate conditions instantaneously. If an element takes 200 milliseconds to appear on screen, standard assertions fail immediately.

Playwright Java introduces **Web-First Assertions** via `PlaywrightAssertions.assertThat()`, which automatically poll and retry assertions until the condition is met or a timeout expires.

---

## 1. Web-First Assertions vs Generic Assertions

```
GENERIC JUNIT ASSERTION (Brittle):
assertTrue(page.locator(".banner").isVisible()); // Fails if banner appears 50ms later!
 
PLAYWRIGHT WEB-FIRST ASSERTION (Robust & Auto-Retrying):
assertThat(page.locator(".banner")).isVisible(); // Automatically polls for up to 5 seconds!
```

---

## 2. Comprehensive Assertions Reference

| Assertion Method | Validation Target |
| :--- | :--- |
| `assertThat(locator).isVisible()` | Element is visible in DOM |
| `assertThat(locator).isEnabled()` | Element is not disabled |
| `assertThat(locator).hasText("Submit")` | Element contains exact/partial text |
| `assertThat(locator).hasValue("John")` | Form input has specific value |
| `assertThat(locator).hasCount(5)` | List contains exactly N elements |
| `assertThat(page).hasURL(".../dashboard")` | Page URL matches expected string/pattern |
| `assertThat(page).hasTitle("MyCodeYatra")` | Page title matches expected title |

---

## 3. Production Test Suite (`src/test/java/com/mycodeyatra/tests/AssertionsTest.java`)

```java
package com.mycodeyatra.tests;
 
import com.microsoft.playwright.*;
import org.junit.jupiter.api.*;
import static com.microsoft.playwright.assertions.PlaywrightAssertions.assertThat;
 
public class AssertionsTest {
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
        page.navigate("https://practice.mycodeyatra.com/form-practice");
    }
 
    @Test
    @DisplayName("Validate Web-First Assertions on Form Practice")
    void testAssertions() {
        assertThat(page.locator("#username")).isEnabled();
        assertThat(page.locator("#username")).isEmpty();
 
        page.fill("#username", "Pankaj Kumar");
        assertThat(page.locator("#username")).hasValue("Pankaj Kumar");
    }
 
    @AfterEach
    void cleanup() {
        page.close();
    }
 
    @AfterAll
    static void teardown() {
        browser.close();
        playwright.close();
    }
}
```

---

## 4. Key Takeaways

1. **Always Use `PlaywrightAssertions.assertThat()`**: Avoid static JUnit/TestNG assertions for DOM elements.
2. **Custom Timeouts**: Pass `new LocatorAssertions.IsVisibleOptions().setTimeout(10000)` for slow network operations.

