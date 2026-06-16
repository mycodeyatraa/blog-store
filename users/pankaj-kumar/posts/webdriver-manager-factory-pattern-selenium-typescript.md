---
title: Selenium Manager & The Factory Design Pattern in TypeScript
date: 21-Nov-2024
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [typescript, selenium, factory-pattern, webdriver-manager, design-patterns]
category: Selenium TypeScript
categories: [Selenium TypeScript, Architecture & Patterns]
excerpt: >-
  Stop manually downloading chromedriver.exe! Learn how native Selenium Manager combined with the Factory Design Pattern creates a frictionless, CI-ready framework.
readTime: 5 min read
---

# Selenium Manager & The Factory Design Pattern in TypeScript

For over a decade, the biggest pain point of UI Automation was **Driver Binary Management**. Every time the Chrome Browser auto-updated on a machine, your automated tests would instantly crash with a `SessionNotCreatedException: This version of ChromeDriver only supports Chrome version X`.

Automation engineers relied on third-party libraries like `webdriver-manager` to constantly fetch the latest binaries.

The good news? **As of Selenium v4.6.0, this is no longer required!** Selenium now ships with an internal tool called "Selenium Manager."

In this tutorial, we will learn how Selenium Manager works natively, and how to wrap it in a robust **Factory Design Pattern** to orchestrate our Enterprise framework.

---

## 1. How Selenium Manager Works

If you execute `new Builder().forBrowser("chrome").build();`, Selenium 4 will intercept the command. 
It will silently check your local machine's Chrome Browser version, automatically download the exact matching `chromedriver.exe` to a hidden cache folder (`~/.cache/selenium`), and launch the browser.

You no longer need to `npm install chromedriver` or manage `PATH` variables!

---

## 2. The Factory Design Pattern

The Factory Pattern is an Object-Oriented concept where a central class (the "Factory") is responsible for instantiating objects (the "WebDrivers").

Why is this useful? Because configuring a WebDriver for a CI/CD pipeline requires a lot of setup (Headless modes, Proxy settings, disabling Sandboxing). If you do this inside your tests, you'll have massive code duplication.

Let's build an `AdvancedDriverFactory` at `tests/utils/advancedDriverFactory.ts`:

```typescript
import { Builder, WebDriver } from "selenium-webdriver";
import chrome from "selenium-webdriver/chrome";
import firefox from "selenium-webdriver/firefox";
export class AdvancedDriverFactory {
  /**
   * Initializes a WebDriver instance based on Environment Variables.
   * Utilizes native Selenium Manager (v4.6.0+) to auto-download binaries!
   */
  static async getDriver(): Promise<WebDriver> {
    const browserName = (process.env.BROWSER || "chrome").toLowerCase();
    const isHeadless = process.env.HEADLESS === "true";
    let builder = new Builder();
    switch (browserName) {
      case "firefox":
        let ffOptions = new firefox.Options();
        if (isHeadless) ffOptions.addArguments("--headless");
        builder.forBrowser("firefox").setFirefoxOptions(ffOptions);
        break;
      case "edge":
        builder.forBrowser("MicrosoftEdge");
        break;
      case "chrome":
      default:
        let chromeOptions = new chrome.Options();
        // Headless execution is required for Jenkins/GitHub Actions
        if (isHeadless) chromeOptions.addArguments("--headless");
        // Additional Chrome tweaks for stability in Linux CI environments
        chromeOptions.addArguments("--no-sandbox");
        chromeOptions.addArguments("--disable-dev-shm-usage");
        builder.forBrowser("chrome").setChromeOptions(chromeOptions);
        break;
    }
    const driver = await builder.build();
    // Only maximize if we have a visible UI
    if (!isHeadless) {
      await driver.manage().window().maximize();
    }
    await driver.manage().setTimeouts({ implicit: 5000 });
    return driver;
  }
}
```

---

## 3. Running the Framework in Headless Mode

By centralizing our WebDriver instantiation into this Factory, we have unlocked incredible flexibility for our DevOps team!

Our entire Jest suite can now be executed completely silently in the background (Headless mode) with a single command:

**Windows PowerShell:**

```bash
> $env:BROWSER="chrome"; $env:HEADLESS="true"; jest
```

**Mac/Linux Bash:**

```bash
> BROWSER=chrome HEADLESS=true jest
```

Because Selenium Manager is running under the hood, the pipeline will never crash due to a missing driver binary!

## Conclusion

The combination of Selenium Manager's native binary handling and the Factory Design Pattern allows you to build a totally frictionless framework. Any developer can clone your repository, type `jest`, and the tests will just magically run—no configuration required.

In our next tutorial, we will explore another architectural masterpiece: **The Builder Pattern for Test Data Generation!**
