---
title: Mastering the Lifecycle: Hooks in pytest-bdd
date: 08-Mar-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [python, selenium, bdd, pytest-bdd, hooks, teardown, test-automation]
category: Selenium Python
categories: [Selenium Python, Behavior Driven Development]
excerpt: >-
  Stop duplicating setup code. Learn how to use pytest_bdd_before_scenario and pytest_bdd_step_error to automatically manage browser sessions and capture failure screenshots.
readTime: 4 min read
---

# Mastering the Lifecycle: Hooks in pytest-bdd

In Behavior Driven Development (BDD), your tests are meant to read like a story: `Given`, `When`, `Then`. 

But behind the scenes of every good story, there is a lot of unseen preparation and cleanup. In automated testing, we need to initialize the WebDriver before the story starts, clear cookies between chapters (scenarios), and close the browser when the story ends.

To manage this hidden lifecycle without cluttering our beautiful Step Definitions, `pytest-bdd` provides a powerful feature known as **Hooks**.

---

## 1. What are pytest-bdd Hooks?

Hooks are special Python functions that are automatically triggered at specific points during the test execution lifecycle. 

Instead of writing `driver = webdriver.Chrome()` at the top of every single test, you write it inside a hook that runs **before** the scenario starts.

The most commonly used `pytest-bdd` hooks are:
- `pytest_bdd_before_scenario`
- `pytest_bdd_after_scenario`
- `pytest_bdd_before_step`
- `pytest_bdd_after_step`
- `pytest_bdd_step_error` (Crucial for failure screenshots!)

These hooks should be placed inside your global `conftest.py` file so they apply to all feature files.

---

## 2. Basic Setup and Teardown Hooks

Let's use hooks to manage our Selenium WebDriver lifecycle. 

In `conftest.py`, we will define a standard `pytest` fixture to yield the driver, but we will also use `pytest-bdd` hooks to print helpful logging information:

```python
import pytest
from selenium import webdriver
# 1. Standard Pytest Fixture for the Driver
@pytest.fixture
def driver():
    # Setup
    options = webdriver.ChromeOptions()
    options.add_argument("--headless")
    driver = webdriver.Chrome(options=options)
    driver.implicitly_wait(10)
    yield driver
    # Teardown
    driver.quit()
# 2. Pytest-BDD Hooks for Logging
def pytest_bdd_before_scenario(request, feature, scenario):
    """
    Runs before every scenario starts.
    """
    print(f"\n[STARTING SCENARIO]: {scenario.name}")
    print(f"[FEATURE]: {feature.name}")
def pytest_bdd_after_scenario(request, feature, scenario):
    """
    Runs after every scenario finishes.
    """
    print(f"\n[FINISHED SCENARIO]: {scenario.name}")
```

Now, every time you run your test suite, you get beautiful console logs declaring exactly which English scenario is currently executing!

---

## 3. The Most Important Hook: `pytest_bdd_step_error`

When a standard Pytest fails, you get a stack trace. But when a BDD test fails, it is often helpful to know exactly *which* English step failed, and more importantly, what the screen looked like at that exact moment.

The `pytest_bdd_step_error` hook is triggered the millisecond a step fails. We can use this to automatically capture a screenshot!

```python
import os
def pytest_bdd_step_error(request, feature, scenario, step, step_func, step_func_args, exception):
    """
    Triggered when a step fails. Captures a screenshot of the failure.
    """
    print(f"\n❌ [FAILED STEP]: {step.keyword} {step.name}")
    print(f"Exception: {exception}")
    # Ensure reports directory exists
    os.makedirs("reports/screenshots", exist_ok=True)
    # The driver fixture is passed inside step_func_args!
    driver = step_func_args.get("driver")
    if driver:
        # Create a safe filename using the scenario and step name
        safe_name = f"{scenario.name}_{step.name}".replace(" ", "_").replace("/", "-")
        screenshot_path = f"reports/screenshots/FAIL_{safe_name}.png"
        # Save screenshot
        driver.save_screenshot(screenshot_path)
        print(f"📸 Screenshot saved to: {screenshot_path}")
```

### Why is this so powerful?
Without this hook, you would have to wrap every single one of your Step Definitions in a `try/except` block to catch errors and take screenshots. By using `pytest_bdd_step_error`, you write the screenshot logic *once* in `conftest.py`, and it universally protects every single step in your entire framework!

---

## 4. Hooking into Steps for Debugging

Sometimes, you need extreme verbosity when debugging a flaky pipeline. You can use the `before_step` and `after_step` hooks to log the exact execution timeline.

```python
def pytest_bdd_before_step(request, feature, scenario, step, step_func):
    """
    Runs right before a step executes.
    """
    print(f"   -> Executing: {step.keyword} {step.name}")
def pytest_bdd_after_step(request, feature, scenario, step, step_func, step_func_args):
    """
    Runs right after a step executes successfully.
    """
    print(f"   ✅ Success: {step.keyword} {step.name}")
```

## Conclusion

Hooks are the secret weapon of Enterprise BDD frameworks. They allow you to completely decouple your infrastructure logic (Browser instantiation, Logging, Screenshot capturing) from your business logic (Step Definitions).

In the next tutorial, we will explore **Scenario Context**, which teaches us how to elegantly share data (like a generated Order ID) between different `Given`, `When`, and `Then` steps!
