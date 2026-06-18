---
title: The Nightly Regression: Running Tests on Schedules
date: 23-Jan-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [typescript, selenium, ci-cd, github-actions, scheduling, cron, slack]
category: Selenium TypeScript
categories: [Selenium TypeScript, CI/CD]
excerpt: >-
  Establish a true Nightly Regression pipeline by mastering GitHub Actions Cron schedules and configuring automated Slack notifications for your engineering team.
readTime: 4 min read
---

# The Nightly Regression: Running Tests on Schedules

If you have successfully implemented cross-browser execution, headless infrastructure, and artifact extraction, you now possess a fully operational Continuous Integration pipeline.

However, running a 5,000-scenario regression suite on every single developer pull request is a fantastic way to bankrupt your company's cloud budget.

Instead, the industry standard is to run "Smoke Tests" on every Pull Request, and reserve the massive, heavy "Regression Suite" for the middle of the night.

We accomplish this using **Scheduled Workflows**.

---

## 1. Mastering Cron Syntax

In GitHub Actions, scheduled workflows are driven by POSIX Cron syntax. 

Cron syntax consists of 5 fields representing: `Minute Hour Day-of-Month Month Day-of-Week`.

- `0 3 * * *`: Runs at 3:00 AM UTC every single day.
- `30 2 * * 1-5`: Runs at 2:30 AM UTC, Monday through Friday (skipping weekends).
- `0 0 1 * *`: Runs at Midnight on the 1st day of every month.

> **CRITICAL WARNING:** GitHub Actions servers operate on **UTC (Coordinated Universal Time)**. If your engineering team is located in New York (EST), 3:00 AM UTC is actually 10:00 PM EST! Always convert your local time to UTC when writing Cron schedules!

---

## 2. The Nightly Regression Workflow

Let's create a dedicated workflow file named `.github/workflows/nightly-regression.yml`:

```yaml
name: Nightly Regression Suite
on:
  schedule:
    # Run at 08:00 UTC (3:00 AM EST) Monday through Friday
    - cron: '0 8 * * 1-5'
  # Always include a manual trigger so QA can run it on demand!
  workflow_dispatch:
jobs:
  regression-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
      - name: Install dependencies
        run: npm ci
      - name: Execute Full Regression
        env:
          ADMIN_PASSWORD: ${{ secrets.ADMIN_PASSWORD }}
        # Notice we are explicitly calling the regression tags!
        run: npm run test:bdd -- --tags "@regression"
      - name: Upload HTML Report
        uses: actions/upload-artifact@v3
        if: always()
        with:
          name: regression-report
          path: reports/
```

---

## 3. Team Notifications (Email & Slack)

A nightly regression suite is useless if nobody looks at the results. We want our developers to wake up, drink their coffee, and immediately see a message in Slack or their Email inbox telling them if the overnight build passed or failed.

We can add a final step to our workflow using a community Action to send a Slack notification!

1. Go to your Slack Workspace and create an "Incoming Webhook" URL.
2. Save that URL as a GitHub Secret named `SLACK_WEBHOOK_URL`.
3. Add this step to the very end of your `nightly-regression.yml`:

```yaml
      - name: Send Slack Notification
        uses: rtCamp/action-slack-notify@v2
        if: always()
        env:
          SLACK_WEBHOOK: ${{ secrets.SLACK_WEBHOOK_URL }}
          SLACK_COLOR: ${{ job.status == 'success' && 'good' || 'danger' }}
          SLACK_MESSAGE: "Nightly Regression Suite completed with status: ${{ job.status }}. Download the HTML report from GitHub Actions!"
          SLACK_TITLE: "Automation Run Results"
```

Because we use the ternary operator `${{ job.status == 'success' && 'good' || 'danger' }}`, the Slack message will be color-coded Green if the tests passed, and Red if the tests failed!

## Conclusion

You have now built the Holy Grail of QA Automation: A comprehensive, headless, cross-browser regression suite that automatically runs while you sleep, safely extracts its own evidence, and sends a color-coded Slack message to the entire engineering team before they even arrive at the office.

This is the power of CI/CD.

We are almost at the finish line. In our next tutorial, we will take a step away from GitHub Actions specifically, and explore a universal technology that underpins all modern DevOps: **Docker**. We will learn how to wrap our entire Selenium TypeScript framework into a portable container!
