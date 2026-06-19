---
title: Generating Comprehensive Accessibility Reports in Selenium Python
date: 23-Feb-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [python, selenium, accessibility, reports, html, allure, axe-core]
category: Selenium Python
categories: [Selenium Python, Accessibility]
excerpt: >-
  Raw JSON is useless to Product Managers. Learn how to parse Axe-Core output into styled HTML reports and attach them directly to your Allure dashboards.
readTime: 4 min read
---

# Generating Comprehensive Accessibility Reports in Selenium Python

Now that we have successfully integrated Axe-Core to scan our pages and automated ARIA and Color Contrast validations, we need a way to present this data.

Finding an accessibility violation in your CI/CD pipeline is only half the battle. If you want developers to actually fix the issues, you need to provide them with highly readable, actionable **Accessibility Reports**.

In this tutorial, we will learn how to extract the raw JSON output from Axe-Core and convert it into beautiful HTML reports using Python!

---

## 1. Extracting Raw JSON from Axe-Core

As we saw in previous tutorials, when Axe-Core runs a scan, it returns a massive Python dictionary (which represents JSON data). 

This dictionary contains four main lists:
1. **`violations`**: Elements that definitively failed the WCAG guidelines.
2. **`passes`**: Elements that successfully passed the checks.
3. **`incomplete`**: Elements that Axe couldn't definitively evaluate (requires manual review).
4. **`inapplicable`**: Rules that did not apply to the current page.

Let's look at how we can save just the violations to a file:

```python
import json
from axe_selenium_python import Axe
from selenium import webdriver
driver = webdriver.Chrome()
driver.get("https://practice.mycodeyatra.com")
axe = Axe(driver)
axe.inject()
results = axe.run()
# Save violations to a raw JSON file
with open("accessibility_violations.json", "w") as file:
    json.dump(results["violations"], file, indent=4)
driver.quit()
```

While `JSON` is great for machines, it is terrible for human developers and Product Managers to read. We need an HTML report!

---

## 2. Using `axe-reports` to Generate HTML

The easiest way to generate an HTML report from Axe results in Python is by using third-party formatting libraries or writing a custom Jinja2 HTML template.

However, the open-source community provides a helpful library called `axe-reports` (primarily used in JS, but Python has wrappers, or we can build our own lightweight HTML generator). 

Since we want full control over our Enterprise framework, let's build a custom HTML Generator using Python's built-in string formatting!

### Building a Custom HTML Reporter

Create `a11y_reporter.py`:

```python
import os
class A11yReporter:
    @staticmethod
    def generate_html_report(violations: list, page_name: str):
        """
        Takes a list of Axe violations and generates a styled HTML report.
        """
        html_content = f\"\"\"
        <html>
        <head>
            <title>Accessibility Report: {page_name}</title>
            <style>
                body {{ font-family: Arial, sans-serif; margin: 40px; background-color: #f8f9fa; }}
                h1 {{ color: #dc3545; }}
                .violation-card {{ background: white; padding: 20px; margin-bottom: 20px; border-radius: 8px; box-shadow: 0 4px 6px rgba(0,0,0,0.1); border-left: 5px solid #dc3545; }}
                .impact-critical {{ color: white; background: #dc3545; padding: 3px 8px; border-radius: 4px; font-weight: bold; }}
                .impact-serious {{ color: white; background: #fd7e14; padding: 3px 8px; border-radius: 4px; font-weight: bold; }}
                code {{ background: #eee; padding: 2px 4px; border-radius: 4px; color: #d63384; }}
            </style>
        </head>
        <body>
            <h1>Accessibility Violations for {page_name}</h1>
            <p>Total Violations Found: <strong>{len(violations)}</strong></p>
        \"\"\"
        for v in violations:
            impact_class = f"impact-{v['impact']}"
            html_content += f\"\"\"
            <div class="violation-card">
                <h2>{v['id']} <span class="{impact_class}">{v['impact'].upper()}</span></h2>
                <p><strong>Description:</strong> {v['description']}</p>
                <p><strong>Help:</strong> <a href="{v['helpUrl']}" target="_blank">{v['help']}</a></p>
                <h3>Failing Elements ({len(v['nodes'])}):</h3>
                <ul>
            \"\"\"
            for node in v['nodes']:
                html_content += f"<li><code>{node['html']}</code><br><small>Target: {node['target']}</small></li>"
            html_content += "</ul></div>"
        html_content += "</body></html>"
        # Ensure reports directory exists
        os.makedirs("reports", exist_ok=True)
        report_path = f"reports/A11y_Report_{page_name}.html"
        with open(report_path, "w", encoding="utf-8") as file:
            file.write(html_content)
        print(f"HTML Report generated at: {report_path}")
```

---

## 3. Integrating the Reporter into Pytest

Now we can integrate our custom `A11yReporter` into our main Test class so that every time a page is scanned, a beautiful HTML report is generated for the developers.

```python
import pytest
from selenium import webdriver
from axe_selenium_python import Axe
from a11y_reporter import A11yReporter
class TestAccessibilityReporting:
    def test_homepage_accessibility(self):
        driver = webdriver.Chrome()
        driver.get("https://practice.mycodeyatra.com")
        axe = Axe(driver)
        axe.inject()
        results = axe.run()
        violations = results.get("violations", [])
        if violations:
            # Generate the HTML Report!
            A11yReporter.generate_html_report(violations, "Homepage")
            # Fail the test
            pytest.fail(f"Found {len(violations)} accessibility violations! See reports/A11y_Report_Homepage.html")
        driver.quit()
```

---

## 4. Attaching Reports to Allure

If your Enterprise framework uses **Allure Reporting**, you can easily attach this HTML file directly to the Allure Dashboard so managers can view it without downloading anything!

```python
import allure
if violations:
    A11yReporter.generate_html_report(violations, "Homepage")
    # Attach HTML to Allure
    allure.attach.file(
        "reports/A11y_Report_Homepage.html", 
        name="Accessibility HTML Report", 
        attachment_type=allure.attachment_type.HTML
    )
    pytest.fail("Accessibility checks failed.")
```

## Conclusion

By transforming raw JSON outputs into styled HTML reports, you make Accessibility metrics readable and actionable. By attaching them to Allure, you give your entire team immediate visibility into WCAG compliance.

In the next tutorial, we will explore **Accessibility CLI Tooling** to learn how we can shift these checks even further left into the developer's terminal!
