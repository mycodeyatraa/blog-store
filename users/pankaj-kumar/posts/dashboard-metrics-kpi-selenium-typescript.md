---
title: Understanding the Numbers: Dashboard Metrics
date: 29-Jan-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [typescript, selenium, reporting, metrics, kpi, dashboards, allure]
category: Selenium TypeScript
categories: [Selenium TypeScript, Reporting]
excerpt: >-
  A dashboard is only as valuable as the metrics it tracks. Learn how to interpret False Positive Rates, execution feedback loops, and the crucial distinction between Failed and Broken automation tests.
readTime: 4 min read
---

# Understanding the Numbers: Dashboard Metrics

If you open an Allure Dashboard, you are immediately greeted by a massive pie chart showing Green (Passed), Red (Failed), and Yellow (Broken) slices.

It looks incredibly impressive. But as a Senior QA Engineer, your job is not just to generate the dashboard; your job is to interpret the data and make engineering decisions based upon it. 

What exactly are we measuring, and what defines a "healthy" automation suite?

---

## 1. The Pass Rate vs. The False Positive Rate

The most obvious metric is the **Pass Rate** (Total Passed / Total Tests).

A 100% Pass Rate is the goal. However, if your Pass Rate suddenly drops to 60%, your immediate question shouldn't be "Who broke the code?". Your immediate question must be: **Are these real failures?**

This brings us to the **False Positive Rate**: The percentage of test failures that occurred *even though the application is working perfectly fine*. 

False positives occur due to:
- Network timeouts
- Selenium synchronization issues (StaleElementReferenceException)
- Cloud infrastructure glitches

If your automation suite has a False Positive Rate of 20%, your developers will stop trusting the dashboard entirely. They will ignore the Slack notifications, assuming "Oh, the tests are just flaky today." This phenomenon is called **Alert Fatigue**, and it is the death of Continuous Integration.

A healthy CI pipeline must maintain a False Positive rate of `< 2%`.

---

## 2. Execution Duration (The Feedback Loop)

When a developer submits a Pull Request, the CI pipeline triggers the automation suite. 

How long does the developer have to wait before they get the Allure report? 10 minutes? 2 hours?

This metric is the **Feedback Loop**. 

If your Regression Suite takes 4 hours to execute, no developer is going to sit at their desk waiting for the result. They will switch tasks, merge the code anyway, and deal with the fallout tomorrow. 

**Rule of Thumb for Execution Duration:**
- **Smoke Tests (on Pull Request):** `< 5 minutes`.
- **Full Regression (Nightly):** `< 45 minutes`.

If your execution takes longer than these thresholds, it is time to invest in Docker parallelization (which we covered in Phase 11!).

---

## 3. Product Coverage vs. Code Coverage

Engineers often confuse these two metrics.

**Code Coverage** measures the percentage of your application's source code (the raw JavaScript/TypeScript files) that was executed during the test run. This is a metric for Unit Tests (like Jest Unit), not Selenium.

**Product Coverage (or Requirements Coverage)** measures the percentage of your Business Requirements (e.g., Jira tickets, User Stories) that have a corresponding automated test.

Cucumber's Behavior-Driven Development architecture makes tracking Product Coverage incredibly easy. Because every test is written in plain English, you can map every single `@scenario` tag directly back to a Jira requirement ID!

---

## 4. The "Broken" vs "Failed" Distinction in Allure

Take a close look at an Allure Dashboard. You will notice it distinguishes between two types of failures:
- **Failed (Red):** The application behaved incorrectly. An assertion failed (e.g., `expect(actualText).to.equal("Success")` evaluated to false). This means you likely found a bug in the software.
- **Broken (Yellow):** The automation framework crashed. This happens when Selenium cannot find an element, or a Hook throws an unhandled exception. This means you have a bug in your *automation code*.

A high number of "Broken" tests is a major red flag indicating poor automation architecture or unstable test environments.

## Conclusion

A dashboard is only as valuable as the metrics it tracks. By monitoring your False Positive Rate, measuring your Feedback Loop, and distinguishing between Product Bugs (Red) and Automation Crashes (Yellow), you transform your reporting from a simple pass/fail list into an actionable engineering compass.

But what do we do when we actually encounter a sea of Red and Yellow?

In our next tutorial, **Failure Analysis**, we will explore systematic strategies for investigating and debugging failed Selenium tests using the data provided in our dashboards!
