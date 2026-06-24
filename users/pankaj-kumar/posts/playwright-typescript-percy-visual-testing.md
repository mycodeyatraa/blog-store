---
title: Percy Visual Testing with Playwright TypeScript
date: 08-Apr-2025
lastUpdated: 08-Apr-2025
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: ["playwright", "typescript", "percy", "visual testing"]
category: UI Automation
categories: ["UI Automation", "Playwright", "TypeScript", "Visual Testing"]
excerpt: >-
  Integrate Percy for robust visual regression testing in Playwright and catch UI bugs efficiently.
readTime: 4 min read
---

Visual bugs are notoriously hard to catch with functional assertions. If a CSS update accidentally hides a button by rendering it white-on-white, standard DOM assertions like `.toBeVisible()` will still pass! This is where visual regression testing shines.

In this tutorial, we will integrate **Percy**, a popular visual testing and review platform by BrowserStack, into a Playwright TypeScript project.

### Prerequisites

First, install the Percy CLI and the Percy Playwright integration package:

```bash
npm install -D @percy/cli @percy/playwright
```

You'll also need a Percy account. Once you create a project on the Percy dashboard, you will be given a `PERCY_TOKEN`. Export this token in your terminal environment before running tests.

### Writing a Percy Visual Test

Integrating Percy into your existing Playwright tests is as simple as importing the `percySnapshot` function and passing the Playwright `page` object.

**File:** `tests/percy-visual.spec.ts`

```typescript
import { test, expect } from '@playwright/test';
import percySnapshot from '@percy/playwright';
 
test.describe('Percy Visual Testing', () => {
 
    test('should validate the practice app home page visually', async ({ page }) => {
        // Navigate to the target practice application
        await page.goto('https://practice.mycodeyatra.com/');
 
        // Wait for a key element to ensure the page has completely rendered
        const header = page.locator('header');
        await expect(header).toBeVisible();
 
        // Take a visual snapshot using Percy
        await percySnapshot(page, 'Practice App - Home Page');
    });
 
    test('should validate the practice app login page visually', async ({ page }) => {
        await page.goto('https://practice.mycodeyatra.com/login');
 
        // Wait for the login form to appear
        const loginForm = page.locator('form');
        await expect(loginForm).toBeVisible();
 
        // Take a visual snapshot of the login screen
        await percySnapshot(page, 'Practice App - Login Page');
    });
 
});
```

### Running the Tests

Because Percy intercepts network requests and captures the DOM state, you must run your Playwright tests *through* the Percy CLI wrapper:

```bash
npx percy exec -- npx playwright test tests/percy-visual.spec.ts
```

### Explanation

1. **`@percy/playwright`**: This package provides the `percySnapshot` function. It captures the DOM snapshot and assets of the current page.
2. **Waiting for Elements**: Always use `expect().toBeVisible()` on key elements before capturing a snapshot. Percy captures the exact state of the DOM at that moment; if the page is still loading, your snapshot might be blank or incomplete!
3. **`percy exec`**: The Percy CLI wrapper boots up a local agent that accepts snapshots from the test runner and efficiently uploads them to the Percy infrastructure for rendering and baseline comparison.

By integrating Percy, you get a powerful dashboard that highlights exact pixel differences whenever your UI changes unexpectedly, preventing visual regressions from reaching your users!
