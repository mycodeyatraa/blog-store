---
title: Matrix Execution: Environment Configuration Management
date: 26-Nov-2024
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [typescript, selenium, configuration, dotenv, matrix-execution]
category: Selenium TypeScript
categories: [Selenium TypeScript, Framework Utilities & Configuration]
excerpt: >-
  Stop hardcoding localhost URLs! Learn how to securely manage QA, UAT, and Production environments dynamically using .env files and dotenv.
readTime: 5 min read
---

# Matrix Execution: Environment Configuration Management

A catastrophic failure in UI automation occurs when hardcoded URLs make their way into the repository. 

Imagine pushing `await driver.get("http://localhost:3000");` to the master branch. The CI/CD pipeline will pull your code, launch the browser, and instantly crash because "localhost" doesn't exist on the Jenkins server!

Similarly, if your suite runs against the **QA environment** during the day, how do you point it to the **UAT environment** before a production release?

In this tutorial, we will implement the `dotenv` library to securely manage multiple environments using a **ConfigManager**.

---

## 1. Creating Environment Files

Instead of hardcoding URLs or Passwords in our TypeScript code, we place them in plain-text `.env` files. You can create multiple files for different stages of deployment.

Create a file named `.env.qa` in the root of your project:

```ini
BASE_URL=https://practice.mycodeyatra.com/#/form-practice
ADMIN_USERNAME=qa_admin
ADMIN_PASSWORD=qa_password_123
API_TIMEOUT=15000
```

*(Note: Ensure you add `*.env` to your `.gitignore` to prevent leaking passwords to GitHub!)*

---

## 2. The ConfigManager Utility

Next, we write a Singleton wrapper class using `dotenv`. This class reads an `ENV_NAME` command-line argument and dynamically loads the correct file.

Create `tests/utils/ConfigManager.ts`:

```typescript
import * as dotenv from 'dotenv';
import * as path from 'path';
export class ConfigManager {
  private static isLoaded = false;
  public static loadConfig(): void {
    if (!this.isLoaded) {
      // Determine which environment to load (default to 'qa')
      const env = process.env.ENV_NAME || 'qa';
      const envPath = path.resolve(process.cwd(), `.env.${env}`);
      console.log(`Loading Configuration for Environment: [${env.toUpperCase()}]`);
      dotenv.config({ path: envPath });
      this.isLoaded = true;
    }
  }
  public static get(key: string): string {
    this.loadConfig(); // Lazy loading
    const value = process.env[key];
    if (!value) {
      throw new Error(`Environment variable ${key} is not set in .env!`);
    }
    return value;
  }
}
```

---

## 3. Injecting Configuration into Tests

Now, we replace all hardcoded strings in our test files with `ConfigManager.get()`.

Create `tests/config_management.test.ts`:

```typescript
import { WebDriver, By, until } from "selenium-webdriver";
import { AdvancedDriverFactory } from "./utils/advancedDriverFactory";
import { ConfigManager } from "./utils/ConfigManager";
import { ElementUtils } from "./utils/ElementUtils";
describe("Phase 4 - Configuration Management", () => {
  let driver: WebDriver;
  let baseUrl: string;
  let adminUser: string;
  beforeAll(async () => {
    // 1. Load variables securely
    baseUrl = ConfigManager.get("BASE_URL");
    adminUser = ConfigManager.get("ADMIN_USERNAME");
    driver = await AdvancedDriverFactory.getDriver();
  });
  afterAll(async () => {
    if (driver) { await driver.quit(); }
  });
  it("Should load URLs and Credentials dynamically", async () => {
    // 2. Navigate dynamically
    await driver.get(baseUrl);
    const nameLocator = By.css("[data-testid='first-name']");
    const emailLocator = By.css("[data-testid='email']");
    const submitBtnLocator = By.css("[data-testid='submit-btn']");
    // 3. Inject dynamic credentials
    await ElementUtils.sendKeys(driver, nameLocator, adminUser);
    await ElementUtils.sendKeys(driver, emailLocator, "admin@mycodeyatra.com");
    await ElementUtils.clickElement(driver, submitBtnLocator);
    const msg = await driver.wait(until.elementLocated(By.css("[data-testid='result-message']")), 5000);
    expect(await msg.getText()).toContain("Form submitted successfully");
  });
});
```

---

## 4. Execution Pipeline

You can now toggle environments from the terminal without editing a single line of code!

**Run against QA:**

```bash
> ENV_NAME=qa jest tests/config_management.test.ts
```

**Run against UAT:**

```bash
> ENV_NAME=uat jest tests/config_management.test.ts
```

## Conclusion

Configuration Management is vital for CI/CD pipelines. By extracting URLs and credentials into `.env` files, you guarantee that your code is strictly separated from your infrastructure data. 

In our next tutorial, we will tackle the final topic of Phase 4: **Implementing a Logging Framework!**
