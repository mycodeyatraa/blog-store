---
title: The Enemy Within: Flaky Test Detection
date: 31-Jan-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [typescript, selenium, reporting, flaky-tests, cucumber, quarantine, ci-cd]
category: Selenium TypeScript
categories: [Selenium TypeScript, Reporting]
excerpt: >-
  Protect the integrity of your CI pipeline by learning how to detect flaky automation tests using automatic retries, and ruthlessly quarantine them using Cucumber tags.
readTime: 4 min read
---

# The Enemy Within: Flaky Test Detection

There is nothing more frustrating in software engineering than a test that fails on Monday, passes on Tuesday, and fails again on Wednesday—all without a single line of code changing.

This is a **Flaky Test**. 

Flaky tests are the enemy of Continuous Integration. If a developer's Pull Request is blocked by a red test, and the QA Engineer says, "Oh, just click rerun, it's flaky," you have lost the trust of the development team. They will start ignoring the dashboard entirely.

---

## 1. What Causes Flakiness?

Before we can detect flakiness, we must understand what causes it.

1. **Race Conditions (Selenium Synchronization):** The test tries to click a button before the JavaScript has finished attaching the event listener. (Solution: Proper Explicit Waits!).
2. **State Leakage:** Test A creates a user in the database. Test B expects the database to be empty. If Test A runs before Test B, Test B fails. (Solution: Complete test isolation; tests must clean up their own data).
3. **Third-Party Outages:** The test relies on an external API (like a payment gateway) that is occasionally slow or offline. (Solution: API Mocking!).

---

## 2. Detecting Flakiness with Retry Mechanisms

The easiest way to detect a flaky test is to use an automatic **Retry Mechanism**.

In Cucumber, we can configure the runner to automatically retry a failed scenario up to `X` times before officially marking it as "Failed" on the dashboard.

Update your `cucumber.js` profile:

```javascript
module.exports = {
  default: `--require-module ts-node/register --require tests/bdd/**/*.ts --format ./reporter.ts --retry 2`
};
```

If a test fails on Attempt 1, but passes on Attempt 2, Cucumber will mark it as a pass, but Allure will flag it with a special "Retried" icon. This is your first indicator of flakiness!

---

## 3. The Danger of Silent Retries

While `--retry 2` keeps your CI pipeline green and your developers happy, it masks the underlying disease. 

If you just let a test silently retry forever, you are ignoring a race condition in your code. Eventually, that race condition might manifest as a real bug for a human user!

You must actively track your retries. Allure provides a **"Trend"** graph specifically for this. If you see a specific scenario continually relying on Attempt 2 to pass, it is officially a Flaky Test.

---

## 4. Quarantining the Infection

When you detect a Flaky Test, you must act brutally and immediately. 

Do not leave it in the main regression suite. Do not say "I'll fix it next sprint." 

You must **Quarantine** it.

In Cucumber, we use tags to quarantine tests. Add a `@quarantine` or `@flaky` tag to the scenario:

```gherkin
  @regression @quarantine
  Scenario: User can upload a massive profile picture
    Given the user is on the profile page
    ...
```

Then, update your GitHub Actions pipeline to explicitly *exclude* quarantined tests:

```yaml
      - name: Execute Full Regression
        # Run regression, but IGNORE anything marked quarantine!
        run: npm run test:bdd -- --tags "@regression and not @quarantine"
```

### Why Quarantine?
By removing the test from the CI pipeline, you ensure that the `main` branch remains 100% stable and trustworthy. Developers can merge with confidence. 

Meanwhile, you create a Jira ticket to investigate and fix the root cause of the flakiness (e.g., adding a better `WebDriverWait`) before removing the `@quarantine` tag and reintroducing it to the suite.

## Conclusion

Flaky tests are inevitable in UI automation, but they do not have to be fatal to your CI/CD pipeline. By utilizing automatic retries for detection, monitoring Allure trends, and ruthlessly quarantining unstable scenarios, you can maintain a 100% reliable dashboard.

But how do we track these metrics over months and years?

In our next tutorial, we will explore **Test Analytics**, moving beyond a single test run to analyze the historical health of our entire automation architecture!
