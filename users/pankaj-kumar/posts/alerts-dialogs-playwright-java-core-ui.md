---
title: Alerts & Dialogs - Playwright Java Core UI
date: 14-Jan-2026
lastUpdated: 14-Jan-2026
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
  Handle JavaScript alerts, confirm dialogs, prompt popups, and modal dialogs natively using page.onDialog() event handlers.
readTime: 9 min read
---

# Alerts & Dialogs - Playwright Java Core UI

JavaScript dialogs (`alert()`, `confirm()`, `prompt()`) freeze browser execution until closed. Unlike Selenium WebDriver which requires switching driver focus via `driver.switchTo().alert()`, Playwright Java automatically dismisses all dialogs by default and provides clean event listeners for customized handling.

---

## 1. Playwright Dialog Event Handling Model

```
        Browser Dialog Triggered (alert / confirm / prompt)
                                 |
                                 v
        Playwright `page.onDialog(dialog -> { ... })`
                                 |
           +---------------------+---------------------+
           |                     |                     |
           v                     v                     v
   dialog.accept()        dialog.dismiss()      dialog.accept("input")
   (Accept Alert/Confirm) (Dismiss Confirm)      (Submit Prompt Text)
```

---

## 2. Dialog Automation Examples

```java
// 1. Accept Alert Dialog
page.onDialog(dialog -> {
    System.out.println("Alert Message: " + dialog.message());
    dialog.accept();
});
page.click("#trigger-alert-btn");
 
// 2. Enter Text in Prompt Dialog
page.onDialog(dialog -> {
    dialog.accept("Pankaj Kumar");
});
page.click("#trigger-prompt-btn");
```

---

## 3. Production Page Object (`src/main/java/com/mycodeyatra/pages/DialogsPage.java`)

```java
package com.mycodeyatra.pages;
 
import com.microsoft.playwright.Page;
import com.microsoft.playwright.Locator;
 
public class DialogsPage {
    private final Page page;
    private final Locator alertBtn;
    private final Locator promptBtn;
 
    public DialogsPage(Page page) {
        this.page = page;
        this.alertBtn = page.locator("#alert-btn");
        this.promptBtn = page.locator("#prompt-btn");
    }
 
    public void navigateToOverlays() {
        page.navigate("https://practice.mycodeyatra.com/overlays");
    }
 
    public void triggerAlertWithAutoAccept() {
        page.onDialog(dialog -> dialog.accept());
        alertBtn.click();
    }
}
```

---

## 4. Key Takeaways

1. **Register Listeners Before Clicking**: Always attach `page.onDialog()` listeners before executing the action that triggers the dialog.
2. **Auto-Dismiss Default**: If no listener is registered, Playwright automatically dismisses dialogs so tests do not hang.

