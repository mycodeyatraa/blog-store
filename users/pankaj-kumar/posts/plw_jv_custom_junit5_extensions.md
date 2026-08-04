---
title: Custom JUnit 5 Extensions - Playwright Java Design Patterns
date: 03-Feb-2026
lastUpdated: 03-Feb-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [playwright, java, junit5, automation, design-patterns, mycodeyatra]
category: Playwright Java Design Patterns
categories: [Playwright Java Design Patterns, Playwright Java, Test Automation]
excerpt: >-
  Build custom JUnit 5 Extensions implementing TestWatcher to capture automatic failure screenshots and traces.
readTime: 9 min read
---

# Custom JUnit 5 Extensions - Playwright Java Design Patterns

JUnit 5 Extensions allow intercepting test execution callbacks. By implementing `TestWatcher` and `BeforeEachCallback`, you can build custom extensions that automatically record video traces, capture screenshots on failure, and log test execution metrics.

---

## 1. Custom TestWatcher Implementation

```java
// 1. Custom JUnit 5 TestWatcher Extension
public class PlaywrightTestWatcher implements TestWatcher {
    @Override
    public void testFailed(ExtensionContext context, Throwable cause) {
        System.out.println("TEST FAILED: " + context.getDisplayName());
        // Capture failure screenshot logic here
    }
 
    @Override
    public void testSuccessful(ExtensionContext context) {
        System.out.println("TEST PASSED: " + context.getDisplayName());
    }
}
 
// 2. Applying Extension to Test Class
@ExtendWith(PlaywrightTestWatcher.class)
public class ExtensionTest {
    @Test
    void testScenario() {
        page.navigate("https://practice.mycodeyatra.com/sandbox");
    }
}
```

---

## 2. Production Page Object (`src/main/java/com/mycodeyatra/pages/design/CustomExtensionPage.java`)

```java
package com.mycodeyatra.pages.design;
 
import com.microsoft.playwright.Page;
 
public class CustomExtensionPage {
    private final Page page;
 
    public CustomExtensionPage(Page page) {
        this.page = page;
    }
 
    public void navigateToSandbox() {
        page.navigate("https://practice.mycodeyatra.com/sandbox");
    }
}
```

---

## 3. Key Takeaways

1. **Declarative Extension Binding**: Bind extensions using `@ExtendWith(MyExtension.class)`.
2. **Automatic Artifact Capture**: Capture DOM snapshots and failure screenshots inside `testFailed()` callbacks.

