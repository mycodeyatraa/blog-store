---
title: "Playwright Architecture"
date: "2025-02-16"
description: "Master the core building blocks of Playwright. Learn how the Browser, BrowserContext, and Page objects interact, and why BrowserContexts make Playwright significantly faster than legacy frameworks."
tags: ["Playwright", "TypeScript", "Architecture", "BrowserContext", "Page"]
---

Welcome to Blog 4 of the **Playwright TypeScript Mastery Series**!

Before we start writing automation scripts, we need to understand the structural hierarchy of Playwright. In Selenium, you instantiate an `InternetExplorerDriver` or `ChromeDriver`, and that single object controls the entire browser. 

Playwright takes a much more sophisticated, multi-layered approach. It breaks the browser down into three distinct tiers: the **Browser**, the **BrowserContext**, and the **Page**. 

Understanding this hierarchy is the key to mastering advanced topics like Parallel Execution, Multi-User Testing, and Authentication State.

### Tier 1: The Browser

At the very top of the hierarchy is the **Browser** object.

When you run a Playwright test, it launches a physical browser instance (Chromium, WebKit, or Firefox). Launching a fresh browser is notoriously slow and consumes a massive amount of system memory.

In older frameworks like Selenium, every single test case launches its own fresh browser instance, which is why a 1,000-test suite can take over an hour to run.

Playwright solves this by **reusing** the single Browser instance across multiple tests!

### Tier 2: The BrowserContext (The Magic Feature)

If Playwright reuses the same Browser for multiple tests, how do we prevent Test A from seeing the cookies, cache, and login state of Test B?

The answer is the **BrowserContext**.

A `BrowserContext` is an isolated, incognito-like environment *inside* the Browser. It has its own unique:
*   Cookies
*   Local Storage
*   Session Storage
*   Cache

Creating a new `BrowserContext` is incredibly fast—it takes less than **10 milliseconds**! 

When you run 10 Playwright tests, Playwright launches **one** heavy Browser, but instantly spins up **10** lightweight `BrowserContexts`. Each test gets its own isolated context. They share no data, no cookies, and no cache. This is the secret to Playwright's blazing execution speed.

### Tier 3: The Page

Inside a `BrowserContext`, you can open a **Page**. 

A `Page` is exactly what it sounds like: a single tab or a popup window. 

*   Every test gets at least one `Page` by default.
*   You can open *multiple* pages (tabs) inside a single `BrowserContext` to test scenarios like clicking a link that opens a new tab.

### The Hierarchy Visualized

To picture the architecture, imagine it like this:

1.  **Browser** (The physical Chrome application)
    *   **BrowserContext 1** (Test A's isolated environment)
        *   **Page 1** (Tab 1 navigating to Home Page)
        *   **Page 2** (Tab 2 opened by clicking a link)
    *   **BrowserContext 2** (Test B's isolated environment)
        *   **Page 1** (Tab 1 navigating to Login Page)

### How Playwright Test Handles This

If you were writing raw Playwright code, you would have to manually instantiate the Browser, Context, and Page like this:

```typescript
import { chromium } from 'playwright';
(async () => {
  // 1. Launch the Browser
  const browser = await chromium.launch();
  // 2. Create an isolated Context
  const context = await browser.newContext();
  // 3. Open a new Page (Tab)
  const page = await context.newPage();
  // 4. Navigate to a URL
  await page.goto('https://practice.mycodeyatra.com/');
  await browser.close();
})();
```

However, because we are using the **Playwright Test Runner** (`@playwright/test`), all of this setup and teardown is handled for you automatically! 

In the Playwright Test runner, your test script simply receives a fully initialized `page` object as a parameter. It has already launched the Browser and created a fresh Context for you behind the scenes.

### Conclusion

The `Browser > BrowserContext > Page` architecture is what makes Playwright the fastest and most reliable testing framework in existence. By isolating state at the Context level instead of the Browser level, Playwright allows you to run completely independent, parallelized tests with virtually zero memory overhead.

In **Blog 5**, we will dive into the **Browser Lifecycle** and learn how to configure the Playwright Test runner to manage the browser launch exactly the way we want it!
