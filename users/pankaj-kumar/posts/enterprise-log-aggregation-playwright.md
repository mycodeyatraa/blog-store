---
title: Enterprise Log Aggregation with Playwright
date: 15-May-2025
lastUpdated: 15-May-2025
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: ["playwright", "typescript", "logging", "elk", "cloudwatch", "observability"]
category: CI/CD & Cloud Execution
categories: ["CI/CD & Cloud Execution", "UI Automation", "Playwright", "TypeScript", "Observability"]
excerpt: >-
  Learn how to serialize Playwright browser console errors and network failures into structured `.jsonl` files for seamless ingestion into ELK, Datadog, or AWS CloudWatch.
readTime: 4 min read
---

When running hundreds of E2E tests in a CI/CD pipeline (or via Docker in the cloud), hunting down a specific JavaScript error inside a 10,000-line terminal output is nearly impossible.

In Enterprise environments, logs must be centralized. **Log Aggregation** is the practice of capturing everything that happens inside the browser and the test runner, formatting it into JSON, and shipping it to an observability platform like **ELK (Elasticsearch/Logstash/Kibana)**, **AWS CloudWatch**, or **Datadog**.

### 1. Intercepting Browser Logs

Playwright can listen to the browser's console using `page.on('console')`. 

Instead of just `console.log()`-ing them into the void of the CI terminal, we can intercept these messages and serialize them into **JSON-Lines (`.jsonl`)** format.

**File:** `tests/log-aggregation.spec.ts`

```typescript
import { test } from '@playwright/test';
import * as fs from 'fs';
 
test('Capture frontend logs for ELK ingestion', async ({ page }) => {
  // 1. Listen for console events
  page.on('console', msg => {
    
    // 2. We only care about warnings and errors, ignore info/debug noise
    if (msg.type() === 'warning' || msg.type() === 'error') {
      
      // 3. Structure the log. CloudWatch and ELK parse JSON natively!
      const logEntry = {
        timestamp: new Date().toISOString(),
        level: msg.type().toUpperCase(),
        text: msg.text(),
        testName: test.info().title,
        location: msg.location()
      };
 
      // 4. Append to a structured JSONL file
      fs.appendFileSync('playwright-logs.jsonl', JSON.stringify(logEntry) + '\n');
    }
  });
 
  await page.goto('/dashboard');
});
```

### 2. How Cloud Ingestion Works

Once your Playwright test generates the `playwright-logs.jsonl` file, how does it get to the dashboard?

You do **not** want your Playwright test to send HTTP `fetch()` requests for every single log. This will slow down your tests and cause rate-limiting.

Instead, you use a **Daemon Agent** (like Filebeat, Fluentd, or the Datadog Agent) running in the background of your CI/CD runner or Docker container.
1. Playwright writes synchronously to the file.
2. The Agent "tails" the file asynchronously.
3. The Agent batches the logs and sends them securely to AWS or ElasticCloud.

### 3. Capturing Network Failures

You shouldn't just capture `console.error`. The most critical logs are failed network requests! You can capture 400/500 level API errors using `page.on('response')`:

```typescript
page.on('response', response => {
  if (response.status() >= 400) {
    const errorLog = {
      level: 'ERROR',
      type: 'NETWORK_FAILURE',
      url: response.url(),
      status: response.status()
    };
    fs.appendFileSync('playwright-logs.jsonl', JSON.stringify(errorLog) + '\n');
  }
});
```

### Summary

By structuring your Playwright browser logs and network failures into a `.jsonl` file, you decouple your automation from your logging infrastructure. Background agents can instantly sweep up your test data, allowing DevOps to search and query frontend errors via Kibana or CloudWatch just like any production microservice!
