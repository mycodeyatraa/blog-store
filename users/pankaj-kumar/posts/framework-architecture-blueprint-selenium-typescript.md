---
title: The Master Blueprint: Architecting an Enterprise Framework
date: 28-Nov-2024
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [typescript, selenium, framework, architecture, basetest]
category: Selenium TypeScript
categories: [Selenium TypeScript, Framework Utilities & Configuration]
excerpt: >-
  Discover how to orchestrate ConfigManagers, Winston Loggers, and Element Utilities into a single BaseTest class to design a truly scalable architecture.
readTime: 6 min read
---

# The Master Blueprint: Architecting an Enterprise Framework

Congratulations on making it to the final tutorial of **Phase 4**!

Over the last few lessons, we have built highly isolated, robust, and scalable components:
1. `AdvancedDriverFactory` (for Browser instantiation)
2. `ConfigManager` (for `.env` loading)
3. `ElementUtils` (for explicit waits and robust actions)
4. `Logger` (for Winston file logging)

However, if we copy/paste the initialization of these 4 modules into the `beforeAll` of 500 test files, we have completely failed at the **DRY (Don't Repeat Yourself)** principle.

To solve this, we will abstract all of this boilerplate into a master **BaseTest** class.

---

## 1. Creating the BaseTest Class

The `BaseTest` acts as the parent container. It automatically hooks into Winston, reads the `.env` file, and instantiates the WebDriver so your actual Test Scripts don't have to.

Create `tests/base/BaseTest.ts`:

```typescript
import { WebDriver } from "selenium-webdriver";
import { AdvancedDriverFactory } from "../utils/advancedDriverFactory";
import { ConfigManager } from "../utils/ConfigManager";
import { Logger } from "../utils/Logger";
export class BaseTest {
  public driver!: WebDriver;
  public baseUrl!: string;
  async setUp(): Promise<void> {
    Logger.info("----------------------------------------");
    Logger.info("TEST EXECUTION STARTED");
    // 1. Load Configurations securely
    this.baseUrl = ConfigManager.get("BASE_URL");
    Logger.info(`Target Environment: ${process.env.ENV_NAME?.toUpperCase() || 'QA'}`);
    // 2. Initialize WebDriver dynamically
    this.driver = await AdvancedDriverFactory.getDriver();
    Logger.info("WebDriver Initialized Successfully.");
  }
  async tearDown(): Promise<void> {
    if (this.driver) {
      Logger.info("Closing Browser and Quitting WebDriver...");
      await this.driver.quit();
    }
    Logger.info("TEST EXECUTION COMPLETED");
    Logger.info("----------------------------------------");
  }
}
```

---

## 2. Refactoring our Test Scripts

Now, observe how incredibly clean and concise our actual test scripts become. We simply instantiate `BaseTest` and call `setUp()`. The test logic is pure, undisturbed automation code.

Create `tests/framework_architecture.test.ts`:

```typescript
import { By, until } from "selenium-webdriver";
import { BaseTest } from "./base/BaseTest";
import { ElementUtils } from "./utils/ElementUtils";
import { Logger } from "./utils/Logger";
describe("Phase 4 - Framework Architecture Blueprint", () => {
  const baseTest = new BaseTest();
  // Let the BaseTest handle all the heavy lifting!
  beforeAll(async () => {
    await baseTest.setUp();
  });
  afterAll(async () => {
    await baseTest.tearDown();
  });
  it("Should orchestrate Config, Utils, Logging, and WebDriver seamlessly", async () => {
    // 1. Navigation using ConfigManager's URL
    Logger.info(`Navigating to ${baseTest.baseUrl}`);
    await baseTest.driver.get(baseTest.baseUrl);
    // 2. Interaction using ElementUtils
    Logger.info("Entering Random Test Data...");
    const nameLocator = By.css("[data-testid='first-name']");
    await ElementUtils.sendKeys(baseTest.driver, nameLocator, "Architect");
    // 3. Assertion
    const val = await baseTest.driver.findElement(nameLocator).getAttribute("value");
    expect(val).toBe("Architect");
    Logger.info("Architecture Validation Passed!");
  });
});
```

---

## 3. Test Execution & The Logs

```bash
> jest tests/framework_architecture.test.ts
```

If we inspect `logs/test-execution.log` after the run:

```text
2026-11-28 12:00:00 [INFO]: ----------------------------------------
2026-11-28 12:00:00 [INFO]: TEST EXECUTION STARTED
2026-11-28 12:00:00 [INFO]: Target Environment: QA
2026-11-28 12:00:03 [INFO]: WebDriver Initialized Successfully.
2026-11-28 12:00:03 [INFO]: Navigating to https://practice.mycodeyatra.com/#/form-practice
2026-11-28 12:00:04 [INFO]: Entering Random Test Data...
2026-11-28 12:00:05 [INFO]: Architecture Validation Passed!
2026-11-28 12:00:05 [INFO]: Closing Browser and Quitting WebDriver...
2026-11-28 12:00:06 [INFO]: TEST EXECUTION COMPLETED
2026-11-28 12:00:06 [INFO]: ----------------------------------------
```

## Conclusion

By implementing a **BaseTest**, we have successfully designed a highly modular, Enterprise-grade Framework Architecture. It is clean, maintainable, and completely abstracted from raw Selenium bindings.

This officially wraps up **Phase 4: Framework Utilities & Configuration!**

In **Phase 5**, we will venture into completely new territory: **API Testing Integration**, where we will combine UI tests with Backend API validations using Axios and SuperTest!
