---
title: Synthetic Monitoring: Using Playwright in CI/CD to Monitor Production
date: 14-May-2025
lastUpdated: 14-May-2025
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: ["playwright", "typescript", "ci-cd", "github-actions", "monitoring", "cloud"]
category: CI/CD & Cloud Execution
categories: ["CI/CD & Cloud Execution", "UI Automation", "Playwright", "TypeScript", "DevOps"]
excerpt: >-
  Upgrade your automation suite into a mission-critical Synthetic Monitoring tool using GitHub Actions Cron jobs, Slack webhooks, and targeted `@smoke` tags.
readTime: 5 min read
---

Playwright is not just a tool for validating Pull Requests. Because it closely mimics real human interactions across multiple browsers, it is the perfect engine for **Synthetic Monitoring**.

Synthetic Monitoring is the practice of running an automated script against your *Live Production Environment* at regular intervals (e.g., every 5 minutes). If the script fails, an alarm fires to page the on-call engineer, alerting them to an outage before a real customer even notices!

We can achieve this using GitHub Actions and Playwright.

### 1. Tagging Non-Destructive Tests

You absolutely **cannot** run your entire E2E suite against Production. Production is real data. If your E2E suite creates test users, submits fake orders, or deletes records, you will corrupt your live system.

First, identify or write a small subset of "read-only" tests (like verifying the homepage loads and the login form appears) and tag them with `@smoke`.

```typescript
import { test, expect } from '@playwright/test';
 
// Notice the @smoke tag!
test('Homepage loads correctly @smoke', async ({ page }) => {
  await page.goto('/');
  await expect(page.locator('text=Welcome')).toBeVisible();
});
```

### 2. Creating the Monitoring Pipeline

Next, we create a GitHub Actions workflow that runs on a cron schedule. It will run `npx playwright test` but filter specifically for the `@smoke` tags!

**File:** `.github/workflows/synthetic-monitoring.yml`

```yaml
name: Production Synthetic Monitoring
 
on:
  schedule:
    - cron: '*/5 * * * *' # Run every 5 minutes
  workflow_dispatch:
 
jobs:
  production-monitor:
    runs-on: ubuntu-latest
 
    steps:
    - uses: actions/checkout@v4
    
    - uses: actions/setup-node@v4
      with:
        node-version: 18
 
    - name: Install dependencies
      run: npm ci
 
    - name: Install Playwright Browsers
      run: npx playwright install --with-deps chromium
 
    - name: Run Production Smoke Tests
      # We ONLY run tests tagged with @smoke.
      run: npx playwright test --grep "@smoke" --project=chromium
      env:
        BASE_URL: https://production.app.com
        CI: true
 
    - name: Alert on Failure
      # This step ONLY runs if the Playwright tests failed!
      if: failure()
      uses: slackapi/slack-github-action@v1.24.0
      with:
        payload: |
          {
            "text": "🚨 *CRITICAL:* Production Monitoring Failed!\n<${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}|View Logs>"
          }
      env:
        SLACK_WEBHOOK_URL: ${{ secrets.SLACK_PROD_ALERTS_WEBHOOK }}
```

### 3. Environment Variables Strategy

When running synthetics against Production, you must strictly manage Environment Variables. Your CI pipeline should inject `BASE_URL=https://production.app.com` into the process, and Playwright should use it dynamically instead of hardcoding environments in the `.spec.ts` files.

### Summary

By combining Playwright's `grep` tagging system with GitHub Actions cron schedules, you can instantly upgrade your test suite into a mission-critical Synthetic Monitoring tool, protecting your production environment 24/7!
