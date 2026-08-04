---
title: Screenshots & Videos - Playwright Java Foundations
date: 11-Jan-2026
lastUpdated: 11-Jan-2026
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
  Configure automatic full-page screenshots, element clips, and video recordings on test failure.
readTime: 8 min read
---

# Screenshots & Videos - Playwright Java Foundations

Capturing visual proof of test execution is vital for debugging regressions and maintaining audit trails. Playwright Java provides native APIs for full-page screenshots, element clips, and automatic video recording.

---

## 1. Full-Page & Element Screenshots

```java
// Full page screenshot
page.screenshot(new Page.ScreenshotOptions()
    .setPath(Paths.get("screenshots/fullpage.png"))
    .setFullPage(true));
 
// Specific element screenshot
page.locator(".card").screenshot(new Locator.ScreenshotOptions()
    .setPath(Paths.get("screenshots/element.png")));
```

---

## 2. Video Recording Configuration

Enable video recording directly when creating a `BrowserContext`:

```java
BrowserContext context = browser.newContext(new Browser.NewContextOptions()
    .setRecordVideoDir(Paths.get("target/videos/"))
    .setRecordVideoSize(1280, 720));
```

---

## 3. Summary

Combining video recording with `PlaywrightAssertions` ensures complete visual observability across your CI/CD test runs.

