---
title: Enforcing Inclusion: Accessibility Testing Basics in Python
date: 13-Feb-2026
lastUpdated: 13-Feb-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: ["selenium", "python", "accessibility", "axe-core", "wcag", "a11y"]
category: Accessibility Testing
categories: ["Accessibility Testing", "Selenium", "Python"]
excerpt: >-
  Shift-left on web accessibility! Learn how to integrate axe-core with Selenium Python to automatically detect WCAG compliance issues during automated test runs.
readTime: 7 min read
---

# Enforcing Inclusion: Accessibility Testing Basics in Python

Digital accessibility ensures that applications remain usable for all individuals, including users with visual, auditory, motor, or cognitive impairments. Integrating Web Content Accessibility Guidelines (**WCAG**) checks directly into automated test suites prevents accessibility regressions before code merges.

---

## 1. Core Accessibility Standards & Rule Engine

Automated accessibility testing relies on rule engines such as **axe-core**. Key evaluation categories include:

- **Perceivable**: Text alternatives for non-text content, color contrast requirements.
- **Operable**: Full keyboard accessibility, focus management, time limits.
- **Understandable**: Clear language attributes, error identification, form labels.
- **Robust**: Valid HTML structure and ARIA attributes for assistive technologies.

---

## 2. Setting Up Axe-Selenium-Python

To begin, install the required packages:

```bash
pip install selenium axe-selenium-python pytest
```

---

## 3. Implementing the Accessibility Test Utility

Create a reusable utility wrapper to run scans and log violations:

**utils/accessibility_helper.py**

```python
import json
from axe_selenium_python import Axe
 
class AccessibilityAuditor:
    def __init__(self, driver):
        self.driver = driver
        self.axe = Axe(driver)
 
    def scan_page(self, report_name: str = "a11y-report.json"):
        # Inject axe-core engine into active DOM
        self.axe.inject()
        results = self.axe.run()
        
        # Save raw results
        self.axe.write_results(results, report_name)
        
        violations = results.get("violations", [])
        return violations
 
    @staticmethod
    def format_violations(violations: list) -> str:
        summary = []
        for index, violation in enumerate(violations, 1):
            summary.append(
                f"{index}. [{violation['impact'].upper()}] {violation['help']} "
                f"(Rule ID: {violation['id']}) - Affected Nodes: {len(violation['nodes'])}"
            )
        return "\n".join(summary)
```

---

## 4. Writing Automated WCAG Compliance Tests

Integrate the auditor directly into your Pytest suite:

**tests/test_accessibility.py**

```python
import pytest
from selenium import webdriver
from utils.accessibility_helper import AccessibilityAuditor
 
@pytest.fixture
def driver():
    driver = webdriver.Chrome()
    driver.maximize_window()
    yield driver
    driver.quit()
 
def test_homepage_accessibility_compliance(driver):
    driver.get("https://mycodeyatra.com")
    
    auditor = AccessibilityAuditor(driver)
    violations = auditor.scan_page("homepage_a11y.json")
    
    # Assert zero critical or serious accessibility violations
    critical_violations = [
        v for v in violations if v.get("impact") in ["critical", "serious"]
    ]
    
    assert len(critical_violations) == 0, (
        f"Accessibility Audit Failed with {len(critical_violations)} issues:\n"
        f"{auditor.format_violations(critical_violations)}"
    )
```

---

## 5. Enterprise Best Practices & CI Integration

1. **Scan Key User Flows**: Run scans post-navigation, after opening modals, and following state changes.
2. **Fail Build on Critical Impact**: Quarantine minor contrast issues while enforcing hard failures on unlabelled form inputs or missing ARIA roles.
3. **Export Artifacts**: Attach HTML/JSON axe reports to CI pipelines (GitHub Actions / Jenkins) for auditing.
