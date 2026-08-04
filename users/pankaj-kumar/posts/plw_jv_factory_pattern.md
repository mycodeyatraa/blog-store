---
title: Factory Pattern - Playwright Java Design Patterns
date: 25-Jan-2026
lastUpdated: 25-Jan-2026
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
  Implement the Factory Creational Pattern for dynamic Browser and BrowserContext instantiation in Playwright Java.
readTime: 9 min read
---

# Factory Pattern - Playwright Java Design Patterns

The Factory Pattern is a creational design pattern that provides a centralized interface for instantiating objects without exposing underlying creation logic to caller classes.

In Playwright Java, a `BrowserFactory` decouples browser initialization flags (headless mode, viewport sizes, proxy settings, channel selection) from test runner classes.

---

## 1. BrowserFactory Architecture

```
                       +-----------------------------------+
                       |         Test Suite Runner         |
                       +-----------------------------------+
                                         |
                       Calls BrowserFactory.getBrowser("chrome")
                                         |
                                         v
                       +-----------------------------------+
                       |          BrowserFactory           |
                       +-----------------------------------+
                        /                |                \
                       v                 v                 v
            Chromium Instance     Firefox Instance   WebKit Instance
```

---

## 2. Implementation Code Example

```java
// 1. Factory Pattern Implementation
public class BrowserFactory {
    public static Browser createBrowser(Playwright playwright, String browserName, boolean headless) {
        BrowserType.LaunchOptions options = new BrowserType.LaunchOptions().setHeadless(headless);
        
        return switch (browserName.toLowerCase()) {
            case "firefox" -> playwright.firefox().launch(options);
            case "webkit" -> playwright.webkit().launch(options);
            default -> playwright.chromium().launch(options);
        };
    }
}
 
// 2. Factory Invocation in Test Setup
@BeforeEach
void setup() {
    playwright = Playwright.create();
    browser = BrowserFactory.createBrowser(playwright, "chromium", true);
    page = browser.newPage();
}
```

---

## 3. Production Page Object (`src/main/java/com/mycodeyatra/pages/design/BrowserFactoryPage.java`)

```java
package com.mycodeyatra.pages.design;
 
import com.microsoft.playwright.Page;
 
public class BrowserFactoryPage {
    private final Page page;
 
    public BrowserFactoryPage(Page page) {
        this.page = page;
    }
 
    public void navigateToSandbox() {
        page.navigate("https://practice.mycodeyatra.com/sandbox");
    }
}
```

---

## 4. Key Takeaways

1. **Centralize Launch Flags**: Encapsulate switches like `--no-sandbox` or `--disable-dev-shm-usage` inside your factory class.
2. **Environment Variable Overrides**: Read target browser parameters directly from `System.getProperty("browser")`.

