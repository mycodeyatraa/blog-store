---
title: Stunning Dashboards: Allure Reporting Basics in Python
date: 15-Feb-2026
lastUpdated: 15-Feb-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: ["selenium", "python", "allure", "reporting", "dashboards", "pytest"]
category: Reporting and Observability
categories: ["Reporting and Observability", "Python", "Automation"]
excerpt: >-
  Transform plain console text logs into visual, executive-ready HTML dashboards! Configure Allure Reporting with Pytest and Selenium Python.
readTime: 8 min read
---

# Stunning Dashboards: Allure Reporting Basics in Python

Command-line output might be sufficient for debugging single test runs, but enterprise stakeholders demand clear visual insights into suite stability, historical failure trends, and exact failure evidence (such as stack traces and screenshots).

**Allure Framework** is an industry-standard, lightweight, multi-language reporting engine that turns test results into interactive HTML dashboards.

---

## 1. Architecture of Allure Reporting

Allure operates in a two-stage process:

1. **Collection Stage**: As Pytest executes, `allure-pytest` writes structured JSON and XML raw data files to a designated directory.
2. **Generation Stage**: The Allure CLI parses raw result files and builds a self-contained, interactive HTML reporting dashboard.

```
 +------------------+                   +----------------------+
 | Pytest Execution | --(writes JSON)-> | allure-results/ dir  |
 +------------------+                   +----------------------+
                                                   |
                                                   v  (allure generate)
                                        +----------------------+
                                        |  Interactive HTML    |
                                        |  Dashboard Report    |
                                        +----------------------+
```

---

## 2. Installation & Prerequisites

Install `allure-pytest` in Python and the Allure CLI tool on your machine:

```bash
# Install Pytest integration plugin
pip install pytest allure-pytest selenium
 
# Install Allure CLI on Windows (Scoop / Chocolatey)
choco install allure
```

---

## 3. Configuring Pytest Hooks & Screenshot Evidence

To automatically capture screenshots and browser logs whenever a test fails, add hooks to `conftest.py`:

**conftest.py**

```python
import pytest
import allure
from selenium import webdriver
 
@pytest.fixture
def driver(request):
    driver = webdriver.Chrome()
    driver.maximize_window()
    yield driver
    driver.quit()
 
# Pytest hook to attach failure evidence to Allure Report
@pytest.hookimpl(tryfirst=True, hookwrapper=True)
def pytest_runtest_makecall(item, call):
    outcome = yield
    rep = outcome.get_result()
    
    if rep.when == "call" and rep.failed:
        driver = item.funcargs.get("driver")
        if driver:
            # Attach Screenshot to Allure
            allure.attach(
                driver.get_screenshot_as_png(),
                name="Failure Screenshot",
                attachment_type=allure.attachment_type.PNG
            )
            # Attach Page Source HTML
            allure.attach(
                driver.page_source,
                name="DOM Dump",
                attachment_type=allure.attachment_type.TEXT
            )
```

---

## 4. Decorating Tests with Feature & Step Annotations

Annotate your Selenium Python tests with metadata to structure the report:

**tests/test_user_profile.py**

```python
import pytest
import allure
from selenium.webdriver.common.by import By
 
@allure.epic("User Management")
@allure.feature("Profile Details")
@allure.story("Update Avatar Image")
@allure.severity(allure.severity_level.CRITICAL)
def test_update_profile_avatar(driver):
    with allure.step("Navigate to User Profile page"):
        driver.get("https://mycodeyatra.com/profile")
 
    with allure.step("Upload new avatar file"):
        upload_input = driver.find_element(By.ID, "avatar-upload")
        upload_input.send_keys("/path/to/image.png")
 
    with allure.step("Click Save Profile button"):
        driver.find_element(By.ID, "save-btn").click()
 
    with allure.step("Verify success toast notification"):
        toast = driver.find_element(By.CLASS_NAME, "toast-message").text
        allure.attach(toast, name="Toast Output", attachment_type=allure.attachment_type.TEXT)
        assert "Profile updated successfully" in toast
```

---

## 5. Generating and Serving Reports

Run your suite to collect raw results, then render the HTML report:

```bash
# 1. Run tests with allure raw results path
pytest tests/ --alluredir=./allure-results
 
# 2. Serve interactive dashboard in local browser
allure serve ./allure-results
```

---

## Best Practices

1. **Use Step Annotations**: Wrap logical browser operations in `with allure.step(...)` for detailed timeline views.
2. **Categorize Severity**: Use `@allure.severity` (BLOCKER, CRITICAL, NORMAL) to prioritize failure triage.
3. **CI Pipeline Integration**: Configure GitHub Actions or Jenkins to store `./allure-report` as a downloadable CI artifact.
