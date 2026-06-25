---
title: Custom Playwright Reporters for Dashboard Metrics
date: 09-May-2025
lastUpdated: 09-May-2025
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: ["playwright", "typescript", "reporting", "metrics", "grafana", "datadog"]
category: Reporting
categories: ["Reporting", "UI Automation", "Playwright", "TypeScript", "Observability"]
excerpt: >-
  Pipe your automation execution metrics directly into Datadog, Grafana, or Slack by building a custom Playwright Reporter class.
readTime: 4 min read
---

While HTML reports and Allure are fantastic for debugging individual test failures, Engineering Leadership usually wants a 10,000-foot view. They don't want to open HTML files; they want to see Pass/Fail rates, execution durations, and flakiness trends plotted over time in enterprise dashboards like **Datadog, Grafana, or New Relic**.

To pipe Playwright data into these dashboards, we need to extract real-time execution metrics. 

Playwright allows you to build **Custom Reporters** using a simple TypeScript class that hooks directly into the test runner's lifecycle!

### 1. Building a Custom Metrics Reporter

Create a new file in your framework to act as your custom reporter. It must implement the Playwright `Reporter` interface.

**File:** `tests/dashboard-reporter.ts`

```typescript
import type { Reporter, TestCase, TestResult, FullResult } from '@playwright/test/reporter';
 
class MetricsDashboardReporter implements Reporter {
  private totalTests = 0;
  private passed = 0;
  private failed = 0;
  private skipped = 0;
  private executionTime = 0;
 
  // 1. Hook into individual test completions
  onTestEnd(test: TestCase, result: TestResult) {
    this.totalTests++;
    if (result.status === 'passed') this.passed++;
    if (result.status === 'failed' || result.status === 'timedOut') this.failed++;
    if (result.status === 'skipped') this.skipped++;
  }
 
  // 2. Hook into the end of the entire test suite execution
  async onEnd(result: FullResult) {
    this.executionTime = result.duration;
 
    // 3. Compile the final metrics payload
    const metricsPayload = {
      timestamp: new Date().toISOString(),
      status: result.status,
      duration_ms: this.executionTime,
      total: this.totalTests,
      passed: this.passed,
      failed: this.failed,
      pass_rate: ((this.passed / this.totalTests) * 100).toFixed(2) + '%'
    };
 
    console.table(metricsPayload);
 
    // 4. Dispatch metrics to an Enterprise Dashboard using fetch()
    /*
    await fetch('https://metrics.enterprise.com/api/v1/ingest', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json', 'Authorization': `Bearer ${process.env.DASHBOARD_TOKEN}` },
      body: JSON.stringify(metricsPayload)
    });
    */
  }
}
 
export default MetricsDashboardReporter;
```

### 2. Registering the Custom Reporter

To activate your new metrics collector, add it to your `playwright.config.ts` file. You can run custom reporters seamlessly alongside your HTML and Allure reporters!

**File:** `playwright.config.ts`

```typescript
import { defineConfig } from '@playwright/test';
 
export default defineConfig({
  reporter: [
    ['html'], 
    ['allure-playwright'],
    ['./tests/dashboard-reporter.ts'] // <-- Point to your custom file!
  ],
});
```

### 3. Alternative: The Built-in JSON Reporter

If you don't want to write a custom TypeScript class, Playwright provides a built-in JSON reporter. This outputs a massive `.json` file containing everything about the test run. 

You can configure your CI/CD pipeline (e.g., a Python script in GitHub Actions) to parse this JSON file and push the metrics to Grafana after Playwright finishes.

```typescript
export default defineConfig({
  reporter: [['json', { outputFile: 'results.json' }]],
});
```

### Summary

Automation data is useless if leadership can't see it. By tapping into Playwright's extensible Reporter interface, you can effortlessly pipe your execution metrics, pass rates, and performance durations directly into Datadog, Grafana, or Slack, providing total observability across your entire enterprise!
