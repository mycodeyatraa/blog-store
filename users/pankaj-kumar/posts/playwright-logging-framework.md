---
title: "Logging Framework Design"
date: "2025-03-14"
description: "Tracing and Console Logs are great, but enterprise frameworks need persistent, structured logging. Learn how to implement a custom logging framework in Playwright."
tags: ["Playwright", "TypeScript", "Logging", "Framework"]
---

Welcome to Blog 30 of the **Playwright TypeScript Mastery Series**!

Playwright has incredibly powerful built-in tracing, video recording, and console logs. However, when you are running a test suite with hundreds of test cases in a CI/CD environment, digging through individual trace files for every minor failure can be exhausting.

Enterprise automation frameworks require a central, persistent text-based log file that captures the step-by-step execution path of the entire suite.

Today, we will build a custom **Logging Utility** to solve this.

### Creating the Logger Utility

We will use the built-in Node.js `fs` (File System) module to create and append to a persistent `.log` file on the disk.

Create a new file `utils/logger.ts`:

```typescript
import fs from 'fs';
import path from 'path';
 
class Logger {
  private logFilePath: string;
 
  constructor() {
    const logDir = path.join(process.cwd(), 'logs');
    if (!fs.existsSync(logDir)) {
      fs.mkdirSync(logDir);
    }
    // Generate a unique log file for this specific execution run
    this.logFilePath = path.join(logDir, `execution-${Date.now()}.log`);
  }
 
  private writeLog(level: string, message: string) {
    const timestamp = new Date().toISOString();
    const logEntry = `[${timestamp}] [${level}] : ${message}\n`;
    
    // Append to file so we have a persistent record
    fs.appendFileSync(this.logFilePath, logEntry);
    
    // Print to terminal so we can see it live
    console.log(logEntry.trim());
  }
 
  info(message: string) {
    this.writeLog('INFO', message);
  }
 
  error(message: string) {
    this.writeLog('ERROR', message);
  }
}
 
// Export as a singleton so all tests write to the exact same file
export default new Logger();
```

### Implementing the Logger in Tests

Now, let's use our custom `Logger` instead of standard `console.log()` inside our tests. By doing this, every step we log is not only printed to the terminal, but permanently saved!

Create `tests/blog30_logging.spec.ts`:

```typescript
import { test, expect } from '@playwright/test';
import Logger from '../utils/logger';
 
test.describe('Blog 30: Logging Framework Design', () => {
 
  test('Using custom logger for test execution', async ({ page }) => {
    Logger.info('Navigating to the practice sandbox...');
    await page.goto('https://practice.mycodeyatra.com/#/sandbox');
    
    Logger.info('Locating the Form Practice card...');
    const formPracticeCard = page.locator('.hover-card').filter({ hasText: 'Form Practice' });
    
    await expect(formPracticeCard).toBeVisible();
    Logger.info('Successfully validated Form Practice card visibility!');
    
    // Simulate logging an error condition
    if (await formPracticeCard.count() === 0) {
      Logger.error('Form Practice card was not found on the page!');
    }
  });
 
});
```

### Execution Output

Run your test using the following command: `npx playwright test tests/blog30_logging.spec.ts`

```
Running 1 test using 1 worker
 
[2026-06-22T12:33:49.551Z] [INFO] : Navigating to the practice sandbox...
[2026-06-22T12:33:49.935Z] [INFO] : Locating the Form Practice card...
[2026-06-22T12:33:50.030Z] [INFO] : Successfully validated Form Practice card visibility!
  OK  1 tests/blog30_logging.spec.ts:6:7 > Blog 30: Logging Framework Design > Using custom logger for test execution (689ms)
 
  1 passed (2.3s)
```

If you look in your project directory, you will now see a `logs/` folder containing an `execution-xyz.log` file with all of your data!

### Conclusion

By building a structured logging utility, you have created a permanent, text-searchable audit trail for your test executions. This is invaluable when debugging transient CI/CD failures where video recordings might be overkill.

In **Blog 31**, we will pull everything together to review **Framework Architecture**, connecting POMs, Fixtures, Configurations, and Utilities into a cohesive structure!
