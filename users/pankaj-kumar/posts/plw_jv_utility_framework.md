---
title: Utility Framework - Playwright Java Design Patterns
date: 29-Jan-2026
lastUpdated: 29-Jan-2026
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
  Build reusable utility wrappers for retries, JavaScript execution, element highlighting, and random data generation.
readTime: 9 min read
---

# Utility Framework - Playwright Java Design Patterns

An enterprise test framework requires robust helper utilities to eliminate repetitive boilerplate code. A well-designed utility layer wraps common operations such as dynamic element highlighting, custom JS execution, and random data generation.

---

## 1. Framework Utility Helpers

```java
// 1. Element Highlighting Utility
public class ElementUtil {
    public static void highlight(Page page, Locator locator) {
        locator.evaluate("el -> el.style.border='3px solid red'");
    }
 
    public static void scrollToView(Locator locator) {
        locator.scrollIntoViewIfNeeded();
    }
}
 
// 2. Random Data Generation Utility
public class DataGenUtil {
    public static String generateRandomEmail() {
        return "user_" + System.currentTimeMillis() + "@mycodeyatra.com";
    }
}
```

---

## 2. Production Page Object (`src/main/java/com/mycodeyatra/pages/design/UtilityWrapperPage.java`)

```java
package com.mycodeyatra.pages.design;
 
import com.microsoft.playwright.Page;
import com.microsoft.playwright.Locator;
 
public class UtilityWrapperPage {
    private final Page page;
    private final Locator searchInput;
 
    public UtilityWrapperPage(Page page) {
        this.page = page;
        this.searchInput = page.locator("#search");
    }
 
    public void navigateAndHighlight() {
        page.navigate("https://practice.mycodeyatra.com/sandbox");
        searchInput.evaluate("el -> el.style.border='2px solid blue'");
    }
}
```

---

## 3. Key Takeaways

1. **Keep Utilities Stateless**: Declare utility methods as `public static` helper functions.
2. **Avoid Re-inventing APIs**: Use Playwright's native locator options before building custom JS wrappers.

