---
title: Shadow DOM - Playwright Java Core UI
date: 19-Jan-2026
lastUpdated: 19-Jan-2026
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
  Pierce open Shadow DOM elements natively using Playwright locators without shadowRoot JavaScript execution.
readTime: 9 min read
---

# Shadow DOM - Playwright Java Core UI

Web Components and Shadow DOM encapsulate DOM trees to prevent CSS leaking. However, encapsulated Shadow Roots hide elements from traditional XPath and CSS selectors in Selenium.

Playwright's CSS selector engine **automatically pierces open Shadow DOM roots** by default.

---

## 1. Native Shadow DOM Piercing

```html
<!-- DOM Structure -->
<user-card id="host">
  #shadow-root (open)
    <button class="edit-btn">Edit Profile</button>
</user-card>
```

In Playwright Java, you locate elements inside Shadow DOM with simple CSS selectors:

```java
// Automatically pierces open shadow root!
page.locator("user-card .edit-btn").click();
 
// Explicit shadow piercing combinator (>>)
page.locator("#host >> button.edit-btn").click();
```

---

## 2. Production Page Object (`src/main/java/com/mycodeyatra/pages/ShadowDomPage.java`)

```java
package com.mycodeyatra.pages;
 
import com.microsoft.playwright.Page;
import com.microsoft.playwright.Locator;
 
public class ShadowDomPage {
    private final Page page;
    private final Locator shadowInput;
 
    public ShadowDomPage(Page page) {
        this.page = page;
        this.shadowInput = page.locator("shadow-host >> #inside-input");
    }
 
    public void navigateToShadowDom() {
        page.navigate("https://practice.mycodeyatra.com/shadow-dom");
    }
 
    public void fillShadowField(String text) {
        shadowInput.fill(text);
    }
}
```

---

## 3. Summary

Playwright eliminates custom `executeScript("return arguments[0].shadowRoot...")` workarounds entirely.

