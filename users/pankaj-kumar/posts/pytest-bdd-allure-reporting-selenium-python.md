---
title: Bridging the Gap: BDD Reporting with Allure in pytest-bdd
date: 17-Mar-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [python, selenium, bdd, pytest-bdd, allure, reporting, html-reports]
category: Selenium Python
categories: [Selenium Python, Behavior Driven Development]
excerpt: >-
  The capstone of Phase 10. Learn how to generate beautiful, interactive HTML reports that showcase every Gherkin step and automatically attach failure screenshots using Allure and pytest-bdd.
readTime: 4 min read
---

# Bridging the Gap: BDD Reporting with Allure in pytest-bdd

The entire philosophy behind Behavior Driven Development (BDD) is bridging the communication gap between technical engineers and non-technical stakeholders (Product Managers, Business Analysts, and Clients).

We write `.feature` files in plain English so that anyone can read them. But if your CI/CD pipeline executes those tests and only outputs a raw Python stack trace in the terminal, you have completely defeated the purpose of BDD!

Stakeholders don't read terminal outputs. They read **Reports**. 

In this capstone tutorial for Phase 10, we will learn how to integrate **Allure Reporting** with `pytest-bdd` to generate beautiful, interactive HTML reports that showcase every single Gherkin step!

---

## 1. Why Allure Reporting?

There are many reporting tools available for Python (like `pytest-html`), but **Allure** is the undisputed king of BDD reporting. 

Unlike standard reports that just say "Test Passed", Allure natively understands Gherkin. It will break down your test into the exact `Given`, `When`, and `Then` steps from your feature file, showing the execution time and status of each individual step!

### Installation

First, install the Allure pytest adapter:

```bash
pip install allure-pytest
```

*(Note: You will also need the Allure Commandline tool installed on your machine or CI server to generate the HTML from the raw results).*

---

## 2. Generating Raw Allure Data

To generate Allure results when running your `pytest-bdd` suite, simply append the `--alluredir` flag to your execution command:

```bash
pytest tests/ --alluredir=allure-results
```

When the tests finish, you will see a new folder named `allure-results` filled with cryptic `.json` files. These files contain the exact metadata for every Gherkin scenario, step, and failure.

---

## 3. Viewing the HTML Report

To translate those JSON files into a beautiful HTML dashboard, use the Allure CLI:

```bash
allure serve allure-results
```

This command will automatically spin up a local web server and open the report in your browser! 

### What will you see?
If you navigate to the "Behaviors" tab in the Allure report, you will see your tests grouped perfectly by `Feature` and `Scenario`. If you click on a scenario, it will expand to show the exact Gherkin steps:

```text
✅ Given I navigate to the login page (1.2s)
✅ When I enter the username "admin" and password "secret123" (0.5s)
❌ Then I should see the message "Welcome, Admin!" (10.1s)
```

If a step fails, Allure highlights it in red, immediately showing the stakeholder exactly where the business requirement failed!

---

## 4. Enhancing the Report with Screenshots

In Blog 6, we learned how to use the `pytest_bdd_step_error` hook to automatically capture a screenshot when a step fails. 

Let's upgrade that hook! Instead of just saving the screenshot to our local hard drive, let's attach it directly to the Allure report so stakeholders can see exactly what the screen looked like when the step failed!

```python
import allure
from allure_commons.types import AttachmentType
def pytest_bdd_step_error(request, feature, scenario, step, step_func, step_func_args, exception):
    """
    Triggered when a step fails. Attaches a screenshot directly to Allure.
    """
    driver = step_func_args.get("driver")
    if driver:
        # Capture the screenshot as raw PNG bytes
        png_bytes = driver.get_screenshot_as_png()
        # Attach the bytes to the Allure report!
        allure.attach(
            png_bytes, 
            name=f"Failure: {step.name}", 
            attachment_type=AttachmentType.PNG
        )
```

Now, when a stakeholder opens the Allure report and expands the failed `Then` step, they will see a clickable image attachment showing the exact state of the UI!

---

## 5. Attaching Logs and Context

You can attach almost anything to an Allure report. If your `@when` step interacts with a backend API, you can attach the raw JSON response to the step!

```python
import allure
import json
@when("I submit a new order")
def submit_order(driver, context):
    driver.find_element("id", "checkout-btn").click()
    order_id = driver.find_element("id", "generated-order-id").text
    # We can attach text/JSON to the current step in the report!
    allure.attach(
        json.dumps({"order_id": order_id, "status": "pending"}, indent=2),
        name="Backend Response",
        attachment_type=allure.attachment_type.JSON
    )
```

## Conclusion: Phase 10 Complete!

Congratulations! You have mastered Behavior Driven Development in Selenium Python! You now know how to write Gherkin feature files, parse Data Tables and Scenario Outlines, manage state with Scenario Context, and generate beautiful Allure reports that bridge the gap between technical and non-technical teams.

We are now ready to begin **Phase 11: Enterprise Validation**, where we will explore advanced data validation techniques, database assertions, and API/UI hybrid testing strategies!
