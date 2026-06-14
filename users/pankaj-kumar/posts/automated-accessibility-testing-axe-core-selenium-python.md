---
title: Automated Accessibility Testing with Axe-Core in Selenium Python
date: 15-Jul-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [python, selenium, accessibility, axe-core, a11y, wcag]
category: Accessibility Testing
categories: [Accessibility Testing, Python, Automation]
excerpt: >-
  Ensure your web application is legally compliant and usable by everyone! Learn how to integrate the axe-selenium-python library to instantly scan your DOM for WCAG accessibility violations like bad color contrast and missing ARIA tags.
readTime: 6 min read
---

# Automated Accessibility Testing with Axe-Core in Selenium Python

In modern web development, **Web Accessibility (a11y)** is no longer optional. Millions of users rely on screen readers and keyboard navigation to use the internet. Furthermore, strict legal frameworks (like the ADA in the US or the European Accessibility Act) mandate that digital products must be accessible. If your e-commerce checkout button has terrible color contrast or is missing an `aria-label`, your company could face a massive lawsuit.

Manual accessibility testing is painfully slow. As Automation Engineers, we need to shift this testing to the left. In this article, we will teach you how to integrate **Axe-Core**—the industry-standard accessibility engine—into your Python Selenium framework!

---

## 1. What is Axe-Core?

Axe-Core is an open-source JavaScript library developed by Deque Systems. It is the exact same engine that powers the Lighthouse accessibility audits in Google Chrome DevTools. 

By injecting the Axe-Core JavaScript into our Selenium browser session, we can programmatically scan the entire DOM for hundreds of WCAG (Web Content Accessibility Guidelines) violations in less than 500 milliseconds.

---

## 2. Setting Up `axe-selenium-python`

First, we need to install the official Python wrapper for the Axe engine.

```bash
pip install axe-selenium-python
```

---

## 3. Writing Your First Accessibility Test

Let's write a Pytest script that navigates to a webpage, injects the Axe-Core engine, runs the scan, and asserts that there are absolutely zero accessibility violations.

**tests/test_accessibility.py**

```python
import pytest
from selenium import webdriver
from axe_selenium_python import Axe
def test_homepage_accessibility():
    # 1. Initialize the browser
    driver = webdriver.Chrome()
    driver.get("https://mycodeyatra.com")
    # 2. Initialize the Axe Engine and Inject it into the page
    axe = Axe(driver)
    axe.inject()
    # 3. Run the Accessibility Audit!
    print("\n[a11y] Scanning DOM for WCAG Violations...")
    results = axe.run()
    # 4. Extract the Violations Array
    violations = results.get('violations', [])
    # 5. Assert that the page is 100% compliant
    if len(violations) > 0:
        print(f"\n[Error] Found {len(violations)} accessibility violations!")
        # Print a readable report of what broke
        print(axe.report(violations))
    assert len(violations) == 0, f"Accessibility tests failed! {len(violations)} violations found."
    driver.quit()
```

If you run this against a poorly designed website, the Pytest console will explode with a beautifully formatted report detailing exactly *what* failed:
- **`color-contrast`**: "Elements must have sufficient color contrast (Target: Button #submit-btn)"
- **`image-alt`**: "Images must have alternate text (Target: img.hero-banner)"
- **`aria-roles`**: "ARIA roles used must conform to valid values"

---

## 4. Excluding Specific Elements or Rules

Sometimes, you have a known accessibility bug in a third-party widget (like a chat bubble) that you cannot fix. You don't want your CI/CD pipeline to fail because of a vendor's bad code.

The `axe.run()` method allows you to exclude specific CSS selectors, or exclude specific rules entirely!

**tests/test_a11y_exclusions.py**

```python
def test_checkout_accessibility_with_exclusions():
    driver = webdriver.Chrome()
    driver.get("https://mycodeyatra.com/checkout")
    axe = Axe(driver)
    axe.inject()
    # Configure the Axe Run Options
    options = {
        # Ignore the third-party chat widget
        "exclude": [["#intercom-chat-widget"]],
        # Turn off the color-contrast rule globally
        "rules": {
            "color-contrast": {"enabled": False}
        }
    }
    results = axe.run(options=options)
    violations = results.get('violations', [])
    assert len(violations) == 0
    driver.quit()
```

---

## 5. Integrating with CI/CD and Reporting

To make accessibility testing truly enterprise-grade, you should save the raw JSON output of the Axe scan and attach it to your Allure or pytest-html reports!

```python
import json
def test_save_a11y_report():
    driver = webdriver.Chrome()
    driver.get("https://mycodeyatra.com")
    axe = Axe(driver)
    axe.inject()
    results = axe.run()
    # Save the full results to a JSON file for the CI/CD artifact storage
    with open("reports/a11y-report.json", "w") as f:
        json.dump(results, f, indent=4)
    assert len(results["violations"]) == 0
    driver.quit()
```
Your DevOps team can now parse `a11y-report.json` to generate beautiful dashboard metrics showing the Accessibility health of your application over time!

## Conclusion

Automated accessibility testing is incredibly easy to implement and provides massive business value. 
- Install `axe-selenium-python`.
- Use `axe.inject()` to load the engine into the browser.
- Use `axe.run()` to scan the DOM for WCAG violations.
- Use the `options` dictionary to exclude known third-party bugs.
- Save the JSON output for your compliance records!

By integrating Axe-Core into your Pytest framework, you ensure that every single code deployment is legally compliant and usable by everyone. 

### 🎉 The Python Selenium Curriculum is 100% Complete!
With this final article, we have covered every single square inch of the UI Automation landscape. Congratulations on reaching the absolute peak of Software Automation Engineering!
