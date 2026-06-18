---
title: Catching the Culprit: Advanced Allure Integrations
date: 27-Jan-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [typescript, selenium, reporting, allure, screenshots, debugging]
category: Selenium TypeScript
categories: [Selenium TypeScript, Reporting]
excerpt: >-
  Supercharge your Allure Dashboard by utilizing Cucumber Hooks to automatically capture and embed Base64 failure screenshots and custom text logs directly into the UI.
readTime: 4 min read
---

# Catching the Culprit: Advanced Allure Integrations

In our previous tutorial, we successfully wired Allure into our Cucumber TypeScript framework. We can now generate a beautiful HTML dashboard that lists our passed and failed scenarios.

However, a red "Failed" badge on a dashboard isn't actually helpful to a developer. They need to know *why* it failed.

In Phase 10, we learned how to use Cucumber Hooks (`@After`) to capture a screenshot when a test failed and save it to a `screenshots/` folder on our computer. 

Today, we are going to take that concept and supercharge it. We are going to take that screenshot and **embed it directly into the Allure HTML Report**, alongside the exact Step Definition where the failure occurred!

---

## 1. The Power of `this.attach()`

In Cucumber, every Step Definition and Hook has access to the isolated Scenario Context (via the `CustomWorld` object we built in Phase 10). 

Cucumber provides a built-in method called `this.attach()`. This method allows us to send raw data (like text logs, JSON, or Base64 images) directly to whatever Reporter is currently listening.

Since we configured Allure as our Reporter, `this.attach()` sends data directly to the Allure Dashboard!

---

## 2. Refactoring the Global After Hook

Let's revisit our `tests/bdd/support/hook.ts` file. 

We need to update our `@After` hook so that if a scenario fails, we take a screenshot and immediately attach it to the report using `this.attach()`.

```typescript
import { After, Status } from '@cucumber/cucumber';
import { CustomWorld } from './world';
After(async function (this: CustomWorld, scenario) {
    // 1. Check if the Scenario failed
    if (scenario.result?.status === Status.FAILED) {
        console.log(`[Cucumber Hook] Scenario "${scenario.pickle.name}" FAILED. Capturing screenshot...`);
        try {
            // 2. Take the screenshot using Selenium
            const screenshot = await this.driver.takeScreenshot();
            // 3. Attach the Base64 image directly to the Allure Report
            this.attach(screenshot, "base64:image/png");
            console.log("[Cucumber Hook] Screenshot successfully attached to Allure!");
        } catch (error) {
            console.error("Failed to take screenshot:", error);
        }
    }
    // Always clean up the browser session!
    if (this.driver) {
        await this.driver.quit();
    }
});
```

*(Note: We use `"base64:image/png"` to explicitly tell Allure how to render the raw string).*

---

## 3. Adding Custom Text Logs

Screenshots are fantastic for UI issues, but what if a test fails because an API request timed out? A screenshot of a loading spinner isn't very helpful.

We can use `this.attach()` to inject plain text logs into the Allure report as well!

Inside a Step Definition (`login.steps.ts`):

```typescript
import { When } from '@cucumber/cucumber';
When('the user clicks the hidden submit button', async function () {
    this.attach("Attempting to locate the hidden submit button...", "text/plain");
    try {
        await this.driver.findElement(By.id("hidden-btn")).click();
        this.attach("Successfully clicked the button!", "text/plain");
    } catch (err) {
        this.attach(`CRITICAL ERROR: Button not found. Stack trace: ${err.message}`, "text/plain");
        throw err; // Re-throw to fail the test!
    }
});
```

---

## 4. Viewing the Advanced Report

Execute your tests and intentionally force a failure (e.g., change an element ID so Selenium can't find it).

```bash
npm run test:bdd
npm run report:generate
npm run report:open
```

When the dashboard opens:
1. Navigate to the **Suites** tab.
2. Click on the failed Scenario.
3. Expand the specific Step that failed.
4. You will see a beautiful **Attachment** section containing your custom text logs and a high-resolution screenshot of the exact moment the test crashed!

## Conclusion

By utilizing `this.attach()`, we have transformed Allure from a simple pass/fail tracker into an exhaustive root-cause analysis engine. Developers no longer have to dig through terminal logs; everything they need to fix a bug is immediately accessible via the dashboard.

But what if you are stuck using Jest, and you can't use Allure? 

In our next tutorial, we will explore an alternative reporting solution specifically designed for the Jest ecosystem: **Jest HTML Reporters**!
