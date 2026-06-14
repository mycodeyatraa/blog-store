---
title: Advanced Allure: Custom Dashboards and Trend Analysis
date: 13-Jun-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [python, selenium, allure, reporting, ci-cd, analytics]
category: Reporting and Observability
categories: [Reporting and Observability, Python, Automation]
excerpt: >-
  Transform flat Pytest output into an Enterprise Dashboard! Learn how to use Allure annotations for Agile categorization, attach browser console logs, and enable CI/CD Trend Analysis.
readTime: 6 min read
---

# Advanced Allure: Custom Dashboards and Trend Analysis

In Phase 9, we introduced `allure-pytest` to generate beautiful, interactive HTML reports for our Selenium automation suite. We learned how to capture screenshots automatically upon failure. 

However, generating a single report for a single test run is just the beginning. In an Enterprise CI/CD environment, stakeholders want to see **Trends** across hundreds of test executions over time. Product Managers want to categorize failures by business features. Developers want deep diagnostic context attached directly to the UI steps.

In this article, we will explore the advanced features of the Allure Reporting Framework, transforming basic Pytest output into a comprehensive DevOps Dashboard!

---

## 1. Organizing Tests with Allure Annotations

If you have 1,000 automated tests, a flat list of results is unreadable. Allure provides Python decorators to logically group your tests into **Epics**, **Features**, and **Stories**, mimicking Agile methodologies.

**tests/test_checkout.py**

```python
import allure
@allure.epic("E-Commerce Portal")
@allure.feature("Checkout Flow")
class TestCheckout:
    @allure.story("Guest Checkout")
    @allure.severity(allure.severity_level.CRITICAL)
    def test_guest_checkout(self):
        # ... test logic ...
        pass
    @allure.story("Registered User Checkout")
    @allure.severity(allure.severity_level.BLOCKER)
    def test_registered_checkout(self):
        # ... test logic ...
        pass
```

### Why do this?
When you generate your Allure report, you can navigate to the **"Behaviors"** tab. Instead of seeing a massive list of Python files, you will see an interactive tree: `E-Commerce Portal > Checkout Flow > Guest Checkout`. This allows Product Managers to instantly see the health of specific business features!

---

## 2. Dynamic Steps and Logging

Sometimes, a single test function can be very long. If it fails on line 45, it might be difficult to know exactly what the test was trying to do. 

Allure allows you to break down a single Pytest function into readable "Steps" that appear sequentially in the UI report.

```python
import allure
from selenium.webdriver.common.by import By
@allure.title("Validate Login Functionality")
def test_login_flow(driver):
    with allure.step("Navigate to Login Page"):
        driver.get("https://mycodeyatra.com/login")
    with allure.step("Enter User Credentials"):
        driver.find_element(By.ID, "username").send_keys("admin")
        driver.find_element(By.ID, "password").send_keys("SecurePass123!")
    with allure.step("Submit the Form"):
        driver.find_element(By.ID, "login-btn").click()
    with allure.step("Verify Dashboard Redirect"):
        assert "dashboard" in driver.current_url
```

If the test fails during the "Verify Dashboard Redirect" step, the Allure UI will collapse the previous successful steps in green and highlight the final step in bright red.

---

## 3. Attaching Deep Diagnostics

We already know how to attach screenshots on failure. But what if a test fails because the backend returned a 500 API error while the Selenium script was interacting with the UI?

A screenshot of a spinning loading wheel won't help the developer. We need to attach the **Browser Console Logs** and **Network Payloads** directly to the Allure report!

**tests/conftest.py**

```python
import pytest
import allure
from selenium import webdriver
@pytest.fixture(scope="function", autouse=True)
def driver_with_diagnostics(request):
    driver = webdriver.Chrome()
    yield driver
    # Check if the test failed!
    if request.node.rep_call.failed:
        # 1. Attach Screenshot
        allure.attach(
            driver.get_screenshot_as_png(),
            name="Failure Screenshot",
            attachment_type=allure.attachment_type.PNG
        )
        # 2. Attach Browser Console Logs (JavaScript Errors)
        browser_logs = driver.get_log('browser')
        allure.attach(
            str(browser_logs),
            name="Browser Console Logs",
            attachment_type=allure.attachment_type.TEXT
        )
        # 3. Attach current Page Source (DOM state)
        allure.attach(
            driver.page_source,
            name="HTML DOM Snapshot",
            attachment_type=allure.attachment_type.HTML
        )
    driver.quit()
```

When a developer clicks on the failed test in Allure, they will see a beautiful panel containing the screenshot, the raw HTML of the page, and any JavaScript errors thrown in the console!

---

## 4. Enabling Trend Analysis in CI/CD

By default, every time you run `pytest --alluredir=reports`, it generates a completely brand new, isolated report. There is no historical data. 

To see historical **Trends** (a beautiful graph showing Pass/Fail rates over the last 30 days), Allure needs access to the `history/` folder from the *previous* test run.

In a CI/CD pipeline (like GitHub Actions or Jenkins), you must use a plugin to manage this automatically. 

**Example GitHub Actions Workflow snippet:**

```yaml
      - name: Run Pytest Tests
        run: pytest --alluredir=allure-results
      - name: Generate Allure Report and Preserve History
        uses: simple-elf/allure-report-action@master
        if: always()
        with:
          allure_results: allure-results
          gh_pages: gh-pages
          allure_report: allure-report
          allure_history: allure-history
```

The GitHub Actions plugin automatically copies the `history/` folder from the previous pipeline run into the new `allure-results` directory before generating the HTML report. 

When you open the new report, the **"Trend"** graph in the Overview tab will spring to life, showing exactly how your suite's stability has improved (or degraded) over time!

## Conclusion

A test automation framework is only as valuable as the reports it generates.
- Use **@allure.epic**, **@allure.feature**, and **@allure.story** to map your tests to Agile business requirements.
- Wrap complex code blocks in `with allure.step():` to make failures easy to trace.
- Use `allure.attach()` to embed Screenshots, Browser Logs, and HTML source code directly into the failure report.
- Configure your CI/CD pipeline to preserve the `history/` directory to unlock beautiful Trend Analysis graphs!

In our next article, we will explore an alternative reporting tool: generating lightweight, single-file HTML reports using **pytest-html**!
