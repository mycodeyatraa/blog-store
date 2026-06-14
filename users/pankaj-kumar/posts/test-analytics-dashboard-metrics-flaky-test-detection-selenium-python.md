---
title: Test Analytics: Dashboard Metrics and Flaky Test Detection
date: 19-Jun-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [python, selenium, analytics, dashboard, flaky-tests, failure-analysis]
category: Reporting and Observability
categories: [Reporting and Observability, Python, Automation]
excerpt: >-
  Provide Engineering Leadership with true observability! Learn how to extract JSON metrics from Pytest, automatically categorize failure types, and mathematically detect flaky tests using external Cloud Dashboards.
readTime: 6 min read
---

# Test Analytics: Dashboard Metrics and Flaky Test Detection

In our previous articles, we learned how to generate beautiful HTML reports using Allure and `pytest-html`. These reports are excellent for a developer debugging a specific failure that happened *today*. 

However, Engineering Managers do not look at daily HTML reports. They look at long-term **Dashboards**. They want to know the answers to complex questions:
- What is our pass rate over the last 6 months?
- How many of our failures are genuine Application Bugs vs Infrastructure Timeouts?
- Which specific Selenium tests are **Flaky** (randomly passing and failing)?

In this article, we will teach you how to extract raw execution metrics from Pytest and integrate them with Enterprise Test Analytics platforms!

---

## 1. The Flaky Test Epidemic

A **Flaky Test** is a test that occasionally fails and occasionally passes, even though absolutely no code was changed in the application.

Flaky tests are the poison of test automation. If an engineer sees a test fail, but knows that clicking "Re-run" will make it pass, they lose all trust in the automation suite. When a *real* bug is caught, they will assume it is just another flaky test and ignore it.

### How to Detect Flaky Tests?
To detect a flaky test, you must analyze its historical execution data. If `test_checkout_flow` passed on Monday, failed on Tuesday, passed on Wednesday, and failed on Thursday, it is highly flaky.

We can track this automatically using a Pytest plugin called `pytest-json-report`. This plugin outputs our entire execution history as structured JSON, which we can pipe into an analytics database!

```bash
pip install pytest-json-report
```

Run your tests:

```bash
pytest tests/ --json-report --json-report-file=reports/metrics.json
```

**reports/metrics.json**

```json
{
  "summary": {
    "passed": 45,
    "failed": 2,
    "duration": 120.5
  },
  "tests": [
    {
      "nodeid": "tests/test_checkout.py::test_checkout_flow",
      "outcome": "failed",
      "setup": {"duration": 2.1},
      "call": {"duration": 15.4, "crash": {"message": "TimeoutException: Element not interactable"}}
    }
  ]
}
```

By pushing this JSON file into an analytics engine (like ElasticSearch or a custom SQL database) after every CI/CD pipeline run, you can build a Grafana dashboard that automatically highlights the tests with the highest variance in their `"outcome"` over a 30-day period!

---

## 2. Failure Analysis: Categorizing Your Bugs

When an Engineering Manager sees "5 Tests Failed" on a dashboard, they need to know *why*. 

If 5 tests failed because the Login button disappeared, that is a **Product Bug**. If 5 tests failed because the Selenium Grid timed out allocating a browser, that is an **Infrastructure Bug**.

We can use Pytest hooks to perform **Failure Analysis** programmatically! By analyzing the exact Exception thrown by Selenium, we can categorize the failure and attach a custom label to it.

**tests/conftest.py**

```python
import pytest
from selenium.common.exceptions import TimeoutException, WebDriverException
# We hook into the report generation phase
@pytest.hookimpl(hookwrapper=True)
def pytest_runtest_makereport(item, call):
    outcome = yield
    report = outcome.get_result()
    if report.when == "call" and report.failed:
        # Check the exception that caused the crash
        exception_info = str(call.excinfo.value)
        failure_category = "Unknown Bug"
        if isinstance(call.excinfo.value, TimeoutException):
            failure_category = "Infrastructure Timeout / Flaky Network"
        elif isinstance(call.excinfo.value, AssertionError):
            failure_category = "Product Bug / Logic Failure"
        elif isinstance(call.excinfo.value, WebDriverException):
            failure_category = "Selenium Grid / Browser Crash"
        # Attach this category to the report object
        report.failure_category = failure_category
        print(f"\n[Analytics] Test {item.name} failed due to: {failure_category}")
```

If you pair this with our JSON reporter from the previous step, you can build a pie chart on your Enterprise Dashboard showing exactly *why* your suite is failing!

---

## 3. Integrating with Cloud Analytics Platforms

Building your own Grafana dashboard and ElasticSearch database takes time. Fortunately, there are dedicated Cloud Test Analytics platforms that integrate natively with Pytest.

**Popular Test Analytics Platforms:**
1. **ReportPortal.io:** An AI-powered dashboard that automatically categorizes failures and flags flaky tests.
2. **Datadog CI Visibility:** Ingests Pytest metrics and correlates them with backend application performance.
3. **Zephyr / Xray (Jira):** Maps automated test executions directly back to Jira user stories.

To integrate Pytest with ReportPortal, for example, you simply install the official agent:

```bash
pip install pytest-reportportal
```

And configure your `pytest.ini`:

```ini
[pytest]
rp_endpoint = https://reportportal.mycodeyatra.com
rp_uuid = your-secret-api-token
rp_project = ecommerce_automation
rp_launch = nightly_regression
```

When you execute your suite, the agent securely streams the execution status, the Selenium Exception stack traces, and our Failure Categories directly into the ReportPortal cloud dashboard in real-time!

## Conclusion

Enterprise automation requires a macroscopic view of your test suite's health.
- Use **`pytest-json-report`** to extract raw, structured execution data.
- Track test outcomes over time to mathematically identify and quarantine **Flaky Tests**.
- Use Pytest hooks to intercept Selenium Exceptions and perform automatic **Failure Analysis** (Categorizing Product Bugs vs Infrastructure Bugs).
- Stream this data into external dashboards (Grafana, ReportPortal, Datadog) to give Engineering Leadership total observability!

In our next and final article for Phase 12, we will pop the hood of Pytest and learn how to write our very own **Custom Pytest Reporter plugin** from scratch!
