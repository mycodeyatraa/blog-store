---
title: Building from Scratch: Custom Reporters
date: 02-Feb-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [typescript, selenium, reporting, cucumber, automation, api]
category: Selenium TypeScript
categories: [Selenium TypeScript, Reporting]
excerpt: >-
  When out-of-the-box solutions fail, learn how to build your own custom reporting tools from scratch using the powerful Cucumber Formatter API.
readTime: 4 min read
---

# Building from Scratch: Custom Reporters

Throughout this phase, we have explored incredible, off-the-shelf reporting tools like Allure and Jest HTML Reporters. 

But what if your company has a highly specific requirement? What if they want a custom JSON payload sent directly to a proprietary internal dashboard the exact millisecond a test fails? Or what if they want a beautifully formatted PDF generated at the end of the suite?

When out-of-the-box tools fail to meet your needs, it is time to build your own. We do this using the **Cucumber Formatter API**.

---

## 1. The Cucumber Event Protocol

When Cucumber executes a test suite, it emits a stream of "events." 
An event is fired when:
- A test run begins.
- A Feature file is read.
- A Scenario starts.
- A Step passes or fails.
- The test run ends.

A Custom Reporter is simply a TypeScript class that "listens" for these events and executes custom code whenever they occur.

---

## 2. Creating a Custom Formatter

Let's build a simple custom reporter that sends a console alert the moment a Scenario fails, and then saves a custom JSON summary when the entire suite finishes.

Create a file named `custom-reporter.ts` in your project:

```typescript
import { Formatter, IFormatterOptions } from '@cucumber/cucumber';
import * as fs from 'fs';
export default class CustomJSONReporter extends Formatter {
    private passCount = 0;
    private failCount = 0;
    private failedScenarios: string[] = [];
    constructor(options: IFormatterOptions) {
        super(options);
        // Listen for when a Scenario finishes
        options.eventBroadcaster.on('envelope', (envelope) => {
            if (envelope.testCaseFinished) {
                this.processTestCaseFinished(envelope.testCaseFinished);
            }
            // Listen for when the ENTIRE test run finishes
            if (envelope.testRunFinished) {
                this.generateFinalReport();
            }
        });
    }
    private processTestCaseFinished(testCaseFinished: any) {
        // Check if the status was 'FAILED'
        if (testCaseFinished.testResult.status === 'FAILED') {
            this.failCount++;
            // Note: In reality, you'd look up the scenario name using the testCaseId
            this.failedScenarios.push(`Failed Scenario ID: ${testCaseFinished.testCaseStartedId}`);
            // You could execute an HTTP POST to Slack right here!
            console.log("🚨 ALERT: A Scenario just failed! 🚨");
        } else if (testCaseFinished.testResult.status === 'PASSED') {
            this.passCount++;
        }
    }
    private generateFinalReport() {
        const report = {
            executionDate: new Date().toISOString(),
            totalTests: this.passCount + this.failCount,
            passed: this.passCount,
            failed: this.failCount,
            failureDetails: this.failedScenarios
        };
        // Write our custom JSON to a file
        fs.writeFileSync('./custom-summary.json', JSON.stringify(report, null, 2));
        console.log("✅ Custom JSON Report Generated!");
    }
}
```

---

## 3. Wiring the Custom Reporter

To use our new reporter, we update our `cucumber.js` profile to point to our custom class instead of Allure (or we can use both!).

```javascript
module.exports = {
  default: `--require-module ts-node/register --require tests/bdd/**/*.ts --format ./custom-reporter.ts`
};
```

When you execute `npm run test:bdd`, Cucumber will intercept the events, increment our counters, and finally output a pristine `custom-summary.json` file to the root of the repository.

---

## 4. The Power of Customization

Why would you write this instead of using Allure?

Because inside that `processTestCaseFinished` method, you have the full power of Node.js. 
- You could use `nodemailer` to send an email immediately.
- You could execute a SQL query to update an internal QA tracking database.
- You could use the `axios` library to trigger an emergency PagerDuty alert.

With the Formatter API, you are no longer restricted by what a 3rd party tool can do. You have absolute control.

## Conclusion

Congratulations! You have just mastered the final layer of Enterprise QA Architecture.

You now possess the ability to write robust TypeScript code, orchestrate Headless Chrome via Selenium WebDriver, write BDD scenarios in Cucumber, wrap it all in Docker, execute it via GitHub Actions pipelines, and generate fully customized analytical dashboards.

You are no longer a manual tester. You are no longer a junior scripter. 

You are a **Senior Automation Architect**.

Thank you for joining me on this incredible 12-Phase journey. Keep coding, keep testing, and I will see you in the next tutorial!
