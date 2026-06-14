---
title: Allure Reporting: Beautiful Test Execution Dashboards in Python
date: 05-May-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [python, selenium, ci-cd, allure, reporting, pytest]
category: CI/CD Pipelines
categories: [CI/CD Pipelines, Python, Automation]
excerpt: >-
  Stop reading console logs! Learn how to integrate the Allure Framework with Pytest to generate beautiful HTML dashboards, organize tests by severity, and automatically capture Selenium screenshots on failure.
readTime: 6 min read
---

# Allure Reporting: Beautiful Test Execution Dashboards in Python

If your automation suite runs on Jenkins and spits out a 5,000-line console log, nobody is going to read it. When tests fail, developers will just ignore the failures because digging through console logs to find the exact line of code that broke is tedious.

To make your automation framework truly valuable to the enterprise, you must provide **beautiful, interactive reports**. The absolute industry standard for this is the **Allure Framework**.

In this article, we will teach you how to integrate the `allure-pytest` plugin to automatically capture HTML dashboards, error stack traces, and Selenium screenshots whenever a test fails!

---

## 1. Installing Allure

First, you need to install the Allure Pytest integration plugin:

```bash
pip install allure-pytest
```

Second, you need the actual Allure Commandline tool installed on your operating system (or CI/CD server) to convert the raw data into a beautiful HTML website.
- **Mac:** `brew install allure`
- **Windows:** `scoop install allure`
- **Linux (CI/CD):** Download the `.tgz` from the official Allure GitHub releases.

---

## 2. Capturing Screenshots on Failure

Allure allows us to attach `.png` files directly to the HTML report. But we don't want to attach a screenshot for every passing test—that would use up too much hard drive space. We *only* want a screenshot if the Pytest assertion fails!

To do this, we write a custom `pytest_runtest_makereport` hook in our `conftest.py` file. This hook listens for test failures and immediately instructs Selenium to take a picture!

**tests/conftest.py**

```python
import pytest
import allure
from allure_commons.types import AttachmentType
from selenium import webdriver
@pytest.fixture(scope="function")
def driver(request):
    driver_instance = webdriver.Chrome()
    # Attach the driver to the Pytest 'request' object so the hook can access it!
    request.node.driver = driver_instance
    yield driver_instance
    driver_instance.quit()
@pytest.hookimpl(tryfirst=True, hookwrapper=True)
def pytest_runtest_makereport(item, call):
    """Automatically capture a screenshot if a test fails!"""
    outcome = yield
    report = outcome.get_result()
    # If the test failed during the execution phase...
    if report.when == "call" and report.failed:
        # Check if the test was using our 'driver' fixture
        driver = getattr(item, "driver", None)
        if driver:
            # Tell Allure to take a screenshot and attach it!
            allure.attach(
                driver.get_screenshot_as_png(),
                name="Failure_Screenshot",
                attachment_type=AttachmentType.PNG
            )
```

---

## 3. Adding Allure Decorators to Tests

To make the HTML report even more readable for Product Managers, we can use Allure decorators to add human-readable titles, descriptions, and severity levels to our Pytest functions.

**tests/test_checkout.py**

```python
import allure
@allure.epic("E-Commerce Portal")
@allure.feature("Checkout System")
@allure.story("Credit Card Processing")
@allure.severity(allure.severity_level.CRITICAL)
@allure.title("Verify Visa cards process successfully")
def test_visa_checkout(driver):
    with allure.step("Navigate to the Checkout Page"):
        driver.get("https://mycodeyatra.com/checkout")
    with allure.step("Enter Visa Card Details"):
        # Intentionally failing the assertion to trigger our screenshot hook!
        title = driver.title
        assert "Success" in title, f"Expected 'Success', but got '{title}'"
```

Notice the `allure.step()` context managers. These will break the HTML report down into clean, readable steps, showing exactly which step the failure occurred on!

---

## 4. Executing and Generating the Report

To tell Pytest to generate Allure data, you pass the `--alluredir` flag pointing to a temporary folder.

```bash
# 1. Run the test suite and save the raw JSON data
pytest tests/ --alluredir=allure-results
```

```text
============================= test session starts ==============================
collected 1 item
tests/test_checkout.py::test_visa_checkout FAILED
=================================== FAILURES ===================================
______________________________ test_visa_checkout ______________________________
...
E       AssertionError: Expected 'Success', but got 'Checkout - MyCodeYatra'
=========================== 1 failed in 4.12s ==================================
```

Now that the raw data is generated, we use the Allure CLI tool to convert it into a beautiful local web server!

```bash
# 2. Convert the raw JSON into an HTML Dashboard and open it!
allure serve allure-results
```

```text
Generating report to temp directory...
Report successfully generated to /tmp/allure-report
Starting web server...
2026-06-14 10:15:30.123:INFO::main: Logging initialized
Server started at <http://127.0.0.1:54321/>. Press <Ctrl+C> to exit
```

## Conclusion

A test suite is only as good as its reporting.
- Stop relying on console logs. Install `allure-pytest`.
- Use Pytest hooks to automatically attach Selenium screenshots to the HTML report the exact millisecond an `AssertionError` is thrown.
- Use `@allure.step()` and `@allure.severity()` to make your Pytest scripts readable by non-technical Product Managers!

In our next and final article for Phase 9, we will learn how to take the results of this Allure report and automatically broadcast them to your company using **Slack/Microsoft Teams Webhooks**!
