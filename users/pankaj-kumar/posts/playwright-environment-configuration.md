---
title: "Environment-Specific Configuration"
date: "2025-03-15"
description: "Learn how to seamlessly switch your Playwright tests between Dev, QA, and Production environments using dotenv and global configuration patterns."
tags: ["Playwright", "TypeScript", "Configuration", "Dotenv"]
---

Welcome to Blog 29 of the **Playwright TypeScript Mastery Series**!

An automation framework is only as good as its flexibility. Hardcoding URLs and credentials like `https://practice.mycodeyatra.com` directly into your page objects or tests makes your suite extremely rigid. 

What happens when your team asks you to run the exact same tests against the Staging environment or Production? 

Today, we will build an environment-switching mechanism using `dotenv`.

### Understanding Dotenv

`dotenv` is a zero-dependency module that loads environment variables from a `.env` file into `process.env`. By swapping which `.env` file we load at runtime, we can instantly switch target environments.

First, install the library:

```
npm install dotenv
```

### Creating Environment Files

In your project, create a new folder called `config/` and add two files: `qa.env` and `prod.env`.

**`config/qa.env`**

```
BASE_URL=https://practice.mycodeyatra.com
API_URL=https://api.practice.mycodeyatra.com
TEST_USER=admin
TEST_PASS=password123
```

**`config/prod.env`**

```
BASE_URL=https://prod.mycodeyatra.com
API_URL=https://api.prod.mycodeyatra.com
TEST_USER=prod_admin
TEST_PASS=prod_secure_123
```

### Implementing the Environment Switcher

We can instruct Playwright to load these files before any tests run.

Create a new test file: `tests/blog33_env.spec.ts`.

```typescript
import { test, expect } from '@playwright/test';
import dotenv from 'dotenv';
import path from 'path';

// Read the TEST_ENV variable from the command line, defaulting to 'qa'
const envName = process.env.TEST_ENV || 'qa';

// Dynamically load the correct .env file based on the variable
dotenv.config({ path: path.resolve(__dirname, `../config/${envName}.env`) });

test.describe('Blog 33: Environment Configuration', () => {

  test('Read values from the current environment', async ({ page }) => {
    // We access our variables via process.env
    const baseUrl = process.env.BASE_URL;
    const testUser = process.env.TEST_USER;

    console.log(`Running test against environment: ${envName.toUpperCase()}`);
    console.log(`Target URL: ${baseUrl}`);
    console.log(`Test User: ${testUser}`);

    expect(baseUrl).toBeDefined();
    
    // We can now dynamically navigate without hardcoded strings!
    if (baseUrl === 'https://practice.mycodeyatra.com') {
      await page.goto(`${baseUrl}/#/login`);
      await expect(page).toHaveTitle(/MyCodeYatra/);
    }
  });

});
```

### Execution Output

By default, the script loads `qa.env` because we provided it as a fallback. 
Run: `npx playwright test tests/blog33_env.spec.ts`

```
Running test against environment: QA
Target URL: https://practice.mycodeyatra.com
Test User: admin
```

If you are using Git Bash or Mac/Linux, you can inject the environment variable on the fly:
`TEST_ENV=prod npx playwright test tests/blog33_env.spec.ts`

*(For Windows PowerShell, use `$env:TEST_ENV="prod"; npx playwright ...`)*

```
Running test against environment: PROD
Target URL: https://prod.mycodeyatra.com
Test User: prod_admin
```

### Conclusion

With `dotenv`, our automation framework is now completely agnostic. It doesn't care *where* it is running, it simply grabs instructions from the active configuration file and executes. 

This is an absolute necessity for integrating with CI/CD pipelines, which we will explore later. In **Blog 34**, we will tackle **Authentication State Sharing**, drastically speeding up your tests by avoiding repeated logins!
