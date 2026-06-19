---
title: Automating Accessibility with Axe-Core in Selenium Python
date: 15-Feb-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [python, selenium, accessibility, axe-core, wcag, pytest, testing]
category: Selenium Python
categories: [Selenium Python, Accessibility]
excerpt: >-
  Stop manually checking for missing alt tags. Learn how to integrate the Axe-Core engine into your Selenium Python framework to instantly scan and assert WCAG accessibility violations.
readTime: 4 min read
---

# Automating Accessibility with Axe-Core in Selenium Python

In the previous tutorial, we discussed the legal and ethical importance of Accessibility (a11y) testing. We learned how to manually inspect elements for ARIA tags and color contrast. 

However, manual accessibility testing is incredibly slow. As an Enterprise Quality Engineer, you cannot afford to manually check every page of your application for WCAG (Web Content Accessibility Guidelines) violations.

Instead, we need to inject the industry-standard accessibility engine—**Axe-Core**—directly into our Selenium Python framework!

---

## 1. What is Axe-Core?

Axe-Core is a blazing-fast, open-source accessibility testing engine created by Deque Systems. It runs entirely inside the browser and programmatically scans the DOM for hundreds of WCAG violations (missing alt tags, incorrect ARIA attributes, poor color contrast, etc.).

By integrating Axe with Selenium, we can instruct the browser to scan itself instantly after navigating to a page.

---

## 2. Installing the Axe-Selenium-Python Library

The open-source community has provided a fantastic wrapper around Axe-Core specifically for Selenium Python.

Install it via `pip`:

```bash
pip install axe-selenium-python
```

---

## 3. Creating the Accessibility Validator

We do not want to hardcode Axe execution logic into every single test file. Let's create an elegant utility class that we can call from anywhere in our framework.

Create `a11y_validator.py`:

```python
import json
from axe_selenium_python import Axe
from selenium.webdriver.remote.webdriver import WebDriver
class A11yValidator:
    """
    Utility class to execute Axe-Core accessibility scans.
    """
    @staticmethod
    def scan_page(driver: WebDriver, page_name: str) -> dict:
        """
        Injects Axe into the page, runs the scan, and returns the results.
        Raises an AssertionError if critical violations are found.
        """
        # 1. Initialize Axe on the current page
        axe = Axe(driver)
        # 2. Inject the axe-core javascript into the browser
        axe.inject()
        # 3. Run the scan
        print(f"Scanning {page_name} for WCAG violations...")
        results = axe.run()
        # 4. Extract violations
        violations = results.get("violations", [])
        if violations:
            # 5. Save the report to disk for debugging
            report_path = f"reports/a11y_{page_name}_violations.json"
            with open(report_path, "w") as f:
                json.dump(violations, f, indent=4)
            print(f"❌ Found {len(violations)} accessibility violations! Report saved to {report_path}")
            # Format a readable error message for Pytest
            error_msg = f"Accessibility violations found on {page_name}:\n"
            for v in violations:
                error_msg += f"- {v['impact'].upper()}: {v['help']} (Nodes: {len(v['nodes'])})\n"
            raise AssertionError(error_msg)
        print(f"✅ {page_name} passed all Axe accessibility checks!")
        return results
```

---

## 4. Writing the Accessibility Test

Now, let's write a standard Pytest using our new `A11yValidator`.

Create `test_accessibility.py`:

```python
import pytest
from selenium import webdriver
from selenium.webdriver.chrome.options import Options
from utils.a11y_validator import A11yValidator
class TestAccessibility:
    @pytest.fixture(autouse=True)
    def setup_teardown(self):
        options = Options()
        options.add_argument("--headless")
        self.driver = webdriver.Chrome(options=options)
        self.driver.implicitly_wait(10)
        yield
        self.driver.quit()
    def test_login_page_accessibility(self):
        # 1. Navigate to the page
        self.driver.get("https://practice.mycodeyatra.com/login")
        # 2. Assert Accessibility using our utility
        # If there are missing alt text tags or contrast issues, this will fail the test!
        A11yValidator.scan_page(self.driver, "Login_Page")
    def test_dashboard_accessibility(self):
        # You can even scan pages after performing UI actions!
        self.driver.get("https://practice.mycodeyatra.com/login")
        self.driver.find_element("id", "username").send_keys("admin")
        self.driver.find_element("id", "password").send_keys("password123")
        self.driver.find_element("id", "loginBtn").click()
        # Wait for dashboard to load (omitted for brevity)
        # Scan the authenticated dashboard state!
        A11yValidator.scan_page(self.driver, "Authenticated_Dashboard")
```

---

## 5. Understanding the Axe Report

If a test fails, our utility writes a `.json` report to disk. This is what an Axe violation looks like:

```json
[
    {
        "id": "color-contrast",
        "impact": "serious",
        "tags": ["cat.color", "wcag2aa", "wcag143"],
        "description": "Ensures the contrast between foreground and background colors meets WCAG 2 AA contrast ratio thresholds",
        "help": "Elements must have sufficient color contrast",
        "nodes": [
            {
                "html": "<button id='submit'>Click Me</button>",
                "target": ["#submit"]
            }
        ]
    }
]
```

This JSON gives your frontend developers the exact HTML element (`#submit`), the exact WCAG guideline that was violated (`wcag143`), and the impact level (`serious`).

## Conclusion

By integrating Axe-Core into your Python framework, you have automated 80% of accessibility testing. Your UI tests are no longer just checking if features "work"—they are simultaneously ensuring that those features are usable by everyone.

In the next tutorial, we will zoom in on one of the most critical accessibility standards: **ARIA (Accessible Rich Internet Applications) Validation**, and learn how to assert complex dynamic states for screen readers!
