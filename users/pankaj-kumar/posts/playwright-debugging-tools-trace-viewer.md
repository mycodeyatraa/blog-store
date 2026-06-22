---
title: "Debugging Tools and Trace Viewer"
date: "2025-02-21"
description: "Stop using console.log() to debug! Discover Playwright's revolutionary Trace Viewer, VS Code Inspector, and learn how to effortlessly debug failing test executions."
tags: ["Playwright", "TypeScript", "Debugging", "Trace Viewer", "Inspector"]
---

Welcome to Blog 9 of the **Playwright TypeScript Mastery Series**!

When you write a test, it will eventually fail. The web application might change, or you might have written a bad locator. 

In legacy frameworks, debugging a failure usually involves staring at a stack trace, taking random screenshots, and inserting `console.log()` or `Thread.sleep()` everywhere. Playwright completely revolutionized the industry by introducing a suite of incredible visual debugging tools.

Today, we will master the Playwright Inspector, UI Mode, and the magnificent Trace Viewer.

### 1. The Ultimate Weapon: UI Mode

As we discussed in Blog 5, UI Mode is the absolute best way to debug a test locally.

Instead of running a standard test execution, run your test with the `--ui` flag:

```
npx playwright test tests/blog8_e2e_login.spec.ts --ui
```

This opens the Playwright Test App. If a test fails, you can visually scrub through the timeline of the execution. You can hover over the exact moment the test failed and inspect the live DOM *at that precise millisecond*. You can even see what network requests were firing when the locator failed!

### 2. The Playwright Inspector

If you want to physically pause the execution of your test and step through it line-by-line (like a traditional developer debugging code), you can use the Playwright Inspector.

Run your test with the `--debug` flag:

```
npx playwright test tests/blog8_e2e_login.spec.ts --debug
```

When you hit Enter, a browser window will open alongside a small Playwright Inspector window.
*   The test will **pause** on the very first line of code.
*   You can click the **Step over** button to execute one line at a time.
*   You can click **Explore** to hover over elements on the web page and Playwright will automatically generate the `getByRole` locator for you!

### 3. The Revolutionary Trace Viewer

UI Mode and the Inspector are great when you are sitting at your computer. But what happens when your test fails on a remote Jenkins server at 3:00 AM? You can't use `--ui` or `--debug` there!

Enter the **Trace Viewer**.

A Trace is a complete recording of your test execution. It captures screenshots, video, DOM snapshots, network traffic, and console logs. It bundles all of this into a single `.zip` file.

To enable Tracing, open your `playwright.config.ts` and locate the `use` block:

```typescript
import { defineConfig } from '@playwright/test';
 
export default defineConfig({
  use: {
    // Collect a trace for every single test execution
    trace: 'on',
    
    // Alternative: Only collect a trace if the test fails (Best for CI/CD)
    // trace: 'retain-on-failure',
  },
});
```

Now, run your test normally (headless):

```
npx playwright test tests/blog8_e2e_login.spec.ts
```

When the test finishes, Playwright will generate a `trace.zip` file in the `test-results` folder. 

To view it, run:

```
npx playwright show-trace test-results/path-to-trace.zip
```

This will open the Trace Viewer in your browser. It looks identical to UI Mode, allowing you to scrub the timeline and inspect the DOM, except you are viewing a *recording* of an execution that happened on a remote server!

### Conclusion

You now have a complete arsenal of debugging tools. Use **UI Mode (`--ui`)** when developing locally, use the **Inspector (`--debug`)** when you need to step through code line-by-line, and rely on the **Trace Viewer** to figure out why tests are failing in your CI/CD pipelines!

In **Blog 10**, we will begin our Deep Dive into Locators by mastering **Chaining and Filtering** strategies to identify complex, nested web elements!
