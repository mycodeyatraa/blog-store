---
title: "Browser Lifecycle"
date: "2025-02-17"
description: "Master Playwright's execution environment. Learn how to toggle between Headless and Headed modes, utilize the UI Mode for debugging, and manage browser resources effectively."
tags: ["Playwright", "TypeScript", "Browser", "Headless", "Configuration"]
---

Welcome to Blog 5 of the **Playwright TypeScript Mastery Series**!

In the previous blog, we learned that the Playwright Test runner automatically launches the **Browser**, creates an isolated **BrowserContext**, and opens a **Page** for you behind the scenes.

But how do you control exactly *how* that browser launches? Do you want to physically see the browser clicking buttons? Do you want it to run silently in the background? Do you want to view a real-time timeline of your test execution?

Today, we will master the Playwright Browser Lifecycle and explore execution modes.

### Headless vs Headed Mode

By default, when you run `npx playwright test`, Playwright executes your tests in **Headless Mode**.

**What is Headless Mode?**
Headless mode means the browser is launched entirely in memory. It has no graphical user interface (GUI). You will not see a Chrome window pop up on your screen.

*   **Pros**: It is blazing fast and consumes very little CPU/RAM. It is mandatory for running tests on cloud servers (like Jenkins or GitHub Actions).
*   **Cons**: You cannot physically watch the test run, making it harder to debug visual issues.

**What is Headed Mode?**
Headed mode launches the physical, visible browser window so you can watch the test interact with the application in real-time.

### How to Run in Headed Mode

You can control the lifecycle directly from the command line. Open your terminal and run:

```
npx playwright test --headed
```

When you hit Enter, you will physically see Chromium (and Firefox/WebKit if configured) pop up on your screen, navigate to the site, and close automatically when the test finishes.

### Configuring Default Behavior

If you always want to run in Headed mode while developing locally, you don't have to type `--headed` every time. You can modify the lifecycle behavior directly in your `playwright.config.ts` file!

Open your `playwright.config.ts` and locate the `use` block. Update it like this:

```typescript
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  // ... other configs ...
  
  use: {
    // Launch the browser in headed mode by default
    headless: false,
    
    // Slow down Playwright's execution by 500ms per action so you can actually watch it!
    launchOptions: {
      slowMo: 500,
    },
  },
});
```

*Note: `slowMo` is incredibly useful for beginners because Playwright executes so fast in headed mode that the test is often over before you can blink!*

### The Ultimate Debugger: UI Mode

Playwright completely redefined the automation industry when they introduced **UI Mode**.

Instead of running a standard terminal command, run this:

```
npx playwright test --ui
```

This will launch a spectacular graphical application built by Microsoft. The UI Mode allows you to:
1.  See all your tests in a neat folder structure.
2.  Click a "Play" button next to any test.
3.  Watch a live DOM snapshot of your test execution.
4.  Hover over the timeline to go "back in time" and inspect the exact state of the web page *before* and *after* a click happened!
5.  View console logs, network requests, and test source code all in one unified dashboard.

Whenever you are writing or debugging a new test, you should always use `--ui` mode!

### Proper Resource Management

Because the Playwright Test runner handles the Browser lifecycle automatically, you do not need to write `await browser.close()` at the end of your test files. 

If a test passes, Playwright closes the Context and Browser cleanly. If a test crashes, Playwright's internal teardown hooks guarantee that the Browser is forcibly killed, preventing "zombie" Chrome processes from eating up your computer's RAM.

### Conclusion

You now have complete control over how Playwright executes your tests. You can run silently in Headless mode for speed, visually in Headed mode with `slowMo` for observation, or use the immensely powerful UI Mode for debugging.

In **Blog 6**, we will finally dive into the code and master **Locators Deep Dive**, learning how to identify and interact with elements on the page!
