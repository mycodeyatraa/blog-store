---
title: Building Real-Time Slack Alerting Reporters in Playwright
date: 13-May-2025
lastUpdated: 13-May-2025
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: ["playwright", "typescript", "reporting", "slack", "alerting", "custom-reporters"]
category: Reporting
categories: ["Reporting", "UI Automation", "Playwright", "TypeScript", "DevOps"]
excerpt: >-
  Transform Playwright into a proactive alerting engine by building a Custom Reporter that dispatches formatted failure notifications directly to Slack or MS Teams.
readTime: 4 min read
---

We previously looked at Custom Reporters for logging raw performance metrics to a Datadog-style dashboard. But what if you want to actively notify your developers the exact moment a critical bug is detected?

You don't need a heavy external plugin for this. Playwright's `Reporter` interface allows us to build a **Custom Slack/Microsoft Teams Alert Reporter** in under 50 lines of code!

### 1. Building a Custom Slack Reporter

By creating a TypeScript class that implements `Reporter`, we can intercept the exact names of failing tests and compile them into a beautifully formatted Slack message.

**File:** `tests/slack-reporter.ts`

```typescript
import type { Reporter, TestCase, TestResult, FullResult } from '@playwright/test/reporter';
 
class SlackAlertReporter implements Reporter {
  private failedTests: string[] = [];
 
  // 1. Intercept every failing test in real-time
  onTestEnd(test: TestCase, result: TestResult) {
    if (result.status === 'failed' || result.status === 'timedOut') {
      this.failedTests.push(`❌ *${test.title}*`);
    }
  }
 
  // 2. Trigger the Webhook when the suite completely finishes
  async onEnd(result: FullResult) {
    // Prevent spam: Only send an alert if tests actually failed!
    if (this.failedTests.length === 0) return;
 
    // Compile the Slack markdown payload
    const slackPayload = {
      text: `🚨 *Playwright Pipeline Alert* 🚨\n\n*Failing Tests:*\n${this.failedTests.join('\n')}\n\n<https://github.com/my-org/repo/actions|View Logs>`,
    };
 
    // Dispatch the payload via a standard fetch request
    /*
    await fetch(process.env.SLACK_WEBHOOK_URL, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(slackPayload)
    });
    */
  }
}
 
export default SlackAlertReporter;
```

### 2. Registering the Custom Reporter

To activate this feature, simply link the new `.ts` file into your Playwright configuration array. Playwright will automatically compile and run it alongside your HTML reports.

**File:** `playwright.config.ts`

```typescript
import { defineConfig } from '@playwright/test';
 
export default defineConfig({
  reporter: [
    ['html'], 
    ['./tests/slack-reporter.ts'] // Inject the custom Slack Reporter!
  ],
});
```

### 3. Going Further: Real-Time vs Batch Alerting

The example above uses a **Batch Alert** approach (waiting for `onEnd` to fire a single message). 

If you have a 4-hour E2E pipeline, you might not want to wait 4 hours to find out the Login test is broken. You can easily modify the reporter to fire a **Real-Time Alert**:

```typescript
  // Fire the webhook IMMEDIATELY inside onTestEnd instead of onEnd
  onTestEnd(test: TestCase, result: TestResult) {
    if (result.status === 'failed') {
      const payload = { text: `🚨 CRITICAL: *${test.title}* just failed!` };
      fetch(process.env.SLACK_WEBHOOK, { method: 'POST', body: JSON.stringify(payload) });
    }
  }
```

### Summary

By writing just a few lines of code, Playwright's `Reporter` interface transforms from a passive log generator into a proactive, real-time alerting engine that pushes failures directly to where your developers already collaborate!
