---
title: Ditch console.log: Enterprise Logging with Winston
date: 27-Nov-2024
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [typescript, selenium, logging, winston, framework]
category: Selenium TypeScript
categories: [Selenium TypeScript, Framework Utilities & Configuration]
excerpt: >-
  When running 500 tests overnight, console.log is useless. Learn how to implement Winston to generate permanent execution and error log files.
readTime: 5 min read
---

# Ditch console.log: Enterprise Logging with Winston

In a small test script, `console.log("Navigating to Page")` is perfectly fine. 

However, when you have 500 tests running simultaneously on a Jenkins server overnight, `console.log` becomes completely useless. How do you differentiate between a casual info statement, a warning, and a critical error? Where do those logs go once the terminal closes?

For Enterprise Automation, we need a **Logging Framework**. In the Node.js ecosystem, **Winston** is the industry standard.

---

## 1. Installing Winston

First, install the library into your project:

```bash
npm install winston
```

---

## 2. Creating the Logger Utility

We will configure Winston to append timestamps to every log entry and output the data to both the Terminal AND a permanent `.log` file.

Create `tests/utils/Logger.ts`:

```typescript
import winston from "winston";
// 1. Define how our logs will look
const logFormat = winston.format.printf(({ level, message, timestamp }) => {
  return `${timestamp} [${level.toUpperCase()}]: ${message}`;
});
export const Logger = winston.createLogger({
  level: "info", // Default log level
  format: winston.format.combine(
    winston.format.timestamp({ format: "YYYY-MM-DD HH:mm:ss" }),
    logFormat
  ),
  transports: [
    // Output to the Console (Terminal)
    new winston.transports.Console(),
    // Output ALL logs to a physical file
    new winston.transports.File({ filename: "logs/test-execution.log" }),
    // Output ONLY errors to a dedicated error file
    new winston.transports.File({ filename: "logs/error.log", level: "error" })
  ]
});
```

---

## 3. Implementing Winston in our Tests

Now, we replace every `console.log` in our framework with `Logger.info`, `Logger.warn`, or `Logger.error`.

Create `tests/logging_framework.test.ts`:

```typescript
import { WebDriver, By, until } from "selenium-webdriver";
import { AdvancedDriverFactory } from "./utils/advancedDriverFactory";
import { Logger } from "./utils/Logger";
describe("Phase 4 - Logging Framework", () => {
  let driver: WebDriver;
  beforeAll(async () => {
    Logger.info("Initializing WebDriver...");
    driver = await AdvancedDriverFactory.getDriver();
  });
  afterAll(async () => {
    if (driver) {
      Logger.info("Quitting WebDriver...");
      await driver.quit();
    }
  });
  it("Should log test execution steps using Winston", async () => {
    Logger.info("Navigating to Practice Site...");
    await driver.get("https://practice.mycodeyatra.com/#/form-practice");
    Logger.info("Waiting for Header Element...");
    const header = await driver.wait(until.elementLocated(By.css("h1")), 5000);
    const text = await header.getText();
    if (text.includes("Practice")) {
      Logger.info(`Successfully found Header: ${text}`);
    } else {
      Logger.warn("Header text did not match expectations.");
    }
    try {
      Logger.info("Attempting to find a non-existent element to trigger an error log...");
      await driver.findElement(By.css("#this-does-not-exist"));
    } catch (error: any) {
      // 🚨 This will automatically be written to error.log!
      Logger.error(`Element not found: ${error.message}`);
    }
  });
});
```

---

## 4. Test Execution & Log Files

When we execute our test:

```bash
> jest tests/logging_framework.test.ts
```

We see beautifully formatted logs in the terminal:

```text
2026-11-27 10:15:30 [INFO]: Initializing WebDriver...
2026-11-27 10:15:33 [INFO]: Navigating to Practice Site...
2026-11-27 10:15:35 [INFO]: Waiting for Header Element...
2026-11-27 10:15:35 [INFO]: Successfully found Header: Form Practice
2026-11-27 10:15:35 [INFO]: Attempting to find a non-existent element to trigger an error log...
2026-11-27 10:15:35 [ERROR]: Element not found: no such element: Unable to locate element...
2026-11-27 10:15:35 [INFO]: Quitting WebDriver...
```

More importantly, if you look inside your project directory, a new `logs/` folder has been generated containing `test-execution.log` and `error.log`. 

If a nightly run fails, you can now send `error.log` directly to the developers!

## Conclusion

Combining **Environment Configs (`dotenv`)**, **Utility Wrappers (`ElementUtils`)**, and **Enterprise Logging (`Winston`)** forms the holy trinity of a mature Automation Framework.

In our next and final tutorial of Phase 4, we will combine all of these concepts into a master **Framework Architecture Blueprint!**
