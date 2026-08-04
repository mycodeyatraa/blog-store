---
title: Windows & Tabs - Playwright Java Core UI
date: 17-Jan-2026
lastUpdated: 17-Jan-2026
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
  Automate multi-tab popups, new windows, and cross-context switching using context.waitForPage().
readTime: 9 min read
---

# Windows & Tabs - Playwright Java Core UI

Modern web applications frequently open external links, OAuth login flows, or printable reports in new browser tabs or windows (`target="_blank"`).

Playwright Java manages new tabs using event-driven `context.waitForPage()` handlers without requiring manual window handle iterations.

---

## 1. New Tab Capture Model

```java
// Listen for new page creation event triggered by a click
Page newTab = context.waitForPage(() -> {
    page.click("#open-new-tab-btn");
});
 
// Bring new tab to front and interact with elements
newTab.bringToFront();
assertThat(newTab.locator("h1")).hasText("New Window Content");
 
// Close new tab and return to main page
newTab.close();
```

---

## 2. Production Page Object (`src/main/java/com/mycodeyatra/pages/MultiTabWindowPage.java`)

```java
package com.mycodeyatra.pages;
 
import com.microsoft.playwright.Page;
import com.microsoft.playwright.BrowserContext;
 
public class MultiTabWindowPage {
    private final Page page;
 
    public MultiTabWindowPage(Page page) {
        this.page = page;
    }
 
    public void navigateToOverlays() {
        page.navigate("https://practice.mycodeyatra.com/overlays");
    }
 
    public Page openExternalTab() {
        return page.context().waitForPage(() -> {
            page.click("#open-external-btn");
        });
    }
}
```

---

## 3. Key Takeaways

1. **Listen on BrowserContext**: Call `context.waitForPage()` or `page.context().waitForPage()`.
2. **Clean Teardown**: Always close popup tabs after test assertions to avoid memory leaks.

