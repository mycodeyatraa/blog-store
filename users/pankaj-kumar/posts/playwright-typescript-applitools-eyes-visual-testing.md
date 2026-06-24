---
title: Visual Testing in Playwright TypeScript with Applitools Eyes
date: 05-Apr-2025
lastUpdated: 05-Apr-2025
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: ["playwright", "typescript", "applitools", "visual testing"]
category: UI Automation
categories: ["UI Automation", "Playwright", "TypeScript", "Visual Testing"]
excerpt: >-
  Learn how to perform AI-powered visual assertions in Playwright TypeScript using Applitools Eyes.
readTime: 6 min read
---

Visual testing ensures that your web application appears exactly as intended to the user, going beyond traditional functional assertions to verify layout, colors, and responsive design.

In this tutorial, we will learn how to integrate **Applitools Eyes** into a Playwright TypeScript framework. Applitools uses AI-powered visual validation to detect visual differences and manage baselines effortlessly.

### Prerequisites

To get started, make sure you install the Applitools Eyes SDK for Playwright:

```bash
npm install -D @applitools/eyes-playwright
```

You will also need an Applitools API key. You can register for a free account at [Applitools](https://applitools.com/) and grab your key.

### Applitools Eyes Integration Code

Here is a complete example of how to use Applitools Eyes with the Playwright test runner. We will test the login page of the MyCodeYatra practice application.

**File:** `tests/applitools-eyes.spec.ts`

```typescript
import { test, expect } from '@playwright/test';
import { Eyes, Target, Configuration, VisualGridRunner, BrowserType, DeviceName, ScreenOrientation } from '@applitools/eyes-playwright';
 
test.describe('Applitools Eyes Visual Testing', () => {
    let eyes: Eyes;
    let runner: VisualGridRunner;
 
    test.beforeAll(async () => {
        // Create a runner with concurrency of 1
        runner = new VisualGridRunner({ testConcurrency: 1 });
 
        // Initialize Eyes
        eyes = new Eyes(runner);
 
        // Initialize configuration
        const configuration = new Configuration();
 
        // Set your Applitools API key (usually retrieved from environment variable)
        configuration.setApiKey(process.env.APPLITOOLS_API_KEY || 'YOUR_APPLITOOLS_API_KEY');
 
        // Add browsers with different viewports
        configuration.addBrowser(1200, 800, BrowserType.CHROME);
        configuration.addBrowser(1200, 800, BrowserType.FIREFOX);
 
        // Add mobile emulation devices in Portrait mode
        configuration.addDeviceEmulation(DeviceName.iPhone_X, ScreenOrientation.PORTRAIT);
 
        // Set the configuration to eyes
        eyes.setConfiguration(configuration);
    });
 
    test('Visual Validation of Login Page', async ({ page }) => {
        // Start the test
        await eyes.open(
            page,
            'MyCodeYatra Practice App',
            'Login Page Visual Test',
            { width: 1200, height: 800 }
        );
 
        // Navigate to the target login page
        await page.goto('https://practice.mycodeyatra.com/login');
 
        // Wait for the form to be visible to ensure the page has loaded
        await page.waitForSelector('form');
 
        // Check the visual state of the whole window
        await eyes.check('Login Window', Target.window().fully());
 
        // End the test
        await eyes.closeAsync();
    });
 
    test.afterAll(async () => {
        // Wait for all visual tests to finish
        const results = await runner.getAllTestResults(false);
        console.log('Applitools Visual Test Results:', results);
    });
});
```

### Explanation

1. **VisualGridRunner**: We use the Ultrafast Grid runner to run visual tests across multiple browsers and viewports rapidly.
2. **Configuration**: The configuration specifies which browsers and devices we want Applitools to test on.
3. **eyes.open**: Initializes a visual test session and starts recording screens.
4. **eyes.check**: Takes a snapshot of the current state of the page (in this case, fully scrolling the window) and sends it to the Applitools server for comparison against the baseline.
5. **eyes.closeAsync**: Closes the session and processes the results.

### Summary

With just a few lines of code, Applitools Eyes supercharges Playwright with cross-browser AI visual assertions. It takes the heavy lifting out of maintaining layout and responsive testing assertions manually!
