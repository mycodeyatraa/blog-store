---
title: Frames & iFrames - Playwright Java Core UI
date: 18-Jan-2026
lastUpdated: 18-Jan-2026
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
  Interact with embedded iFrames and nested frame structures using frameLocator() without driver context switching.
readTime: 9 min read
---

# Frames & iFrames - Playwright Java Core UI

Automating iFrames (such as payment gateways, reCAPTCHA widgets, or embedded video players) was historically error-prone in Selenium due to manual `driver.switchTo().frame()` state switching.

Playwright Java introduces **Frame Locators**, allowing seamless inline querying inside iFrames.

---

## 1. FrameLocator vs Legacy Frame Switching

```java
// LEGACY SELENIUM (Stateful & Fragile):
driver.switchTo().frame("payment-frame");
driver.findElement(By.ID, "card-num").sendKeys("4111...");
driver.switchTo().defaultContent(); // Must switch back manually!
 
// PLAYWRIGHT JAVA (Declarative & Auto-Waiting):
FrameLocator frame = page.frameLocator("#payment-frame");
frame.locator("#card-num").fill("4111...");
```

---

## 2. Production Page Object (`src/main/java/com/mycodeyatra/pages/FramesPage.java`)

```java
package com.mycodeyatra.pages;
 
import com.microsoft.playwright.Page;
import com.microsoft.playwright.FrameLocator;
import com.microsoft.playwright.Locator;
 
public class FramesPage {
    private final Page page;
    private final FrameLocator singleFrame;
 
    public FramesPage(Page page) {
        this.page = page;
        this.singleFrame = page.frameLocator("#single-frame");
    }
 
    public void navigateToFrames() {
        page.navigate("https://practice.mycodeyatra.com/frames");
    }
 
    public void fillFrameInput(String text) {
        singleFrame.locator("#frame-input").fill(text);
    }
 
    public Locator getFrameHeading() {
        return singleFrame.locator("h2");
    }
}
```

---

## 3. Key Takeaways

1. **Nested iFrames**: Chain `frameLocator()` calls cleanly: `page.frameLocator("#outer").frameLocator("#inner").locator("button")`.
2. **Auto-Waiting**: Frame locators automatically wait for the iFrame to load and attach to DOM.

