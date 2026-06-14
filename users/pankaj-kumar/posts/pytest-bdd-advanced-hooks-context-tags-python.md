---
title: Advanced pytest-bdd: Hooks, Scenario Context, and Tags
date: 19-May-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [python, selenium, bdd, pytest-bdd, hooks, context, tags]
category: Behavior Driven Development
categories: [Behavior Driven Development, Python, Automation]
excerpt: >-
  Elevate your BDD automation to enterprise-grade! Learn how to share data between Given/When/Then steps using Context, filter execution with Tags, and automatically capture screenshots on failure using pytest-bdd Hooks.
readTime: 6 min read
---

# Advanced pytest-bdd: Hooks, Scenario Context, and Tags

In our previous article, we successfully mapped basic Gherkin steps to Selenium WebDriver code using `pytest-bdd`. However, enterprise testing requires more advanced capabilities. 

How do we pass an Order ID from a `When` step to a `Then` step? How do we run only our critical Smoke tests? And how do we automatically capture a screenshot if a specific BDD step fails?

In this article, we will elevate your BDD skills by mastering **Scenario Context**, **Tags**, and **Hooks**.

---

## 1. Scenario Context: Sharing State Between Steps

By default, every step definition function in `pytest-bdd` is completely isolated. But what happens when you generate an Order ID in step 2, and you need to assert that exact Order ID in step 3?

We solve this using **Scenario Context**—a shared dictionary injected via Pytest fixtures!

**tests/features/shopping.feature**

```gherkin
Feature: Shopping Cart
  Scenario: Purchase generates a valid order number
    Given the user adds an item to the cart
    When the user completes the checkout process
    Then the confirmation page should display the generated order number
```

**tests/step_defs/test_shopping_steps.py**

```python
import pytest
from pytest_bdd import scenarios, given, when, then
from selenium.webdriver.common.by import By
scenarios('../features/shopping.feature')
# 1. Create a dictionary fixture to hold our shared context!
@pytest.fixture
def context():
    return {}
@given('the user adds an item to the cart')
def add_item(driver):
    driver.find_element(By.ID, "add-to-cart-btn").click()
@when('the user completes the checkout process')
def complete_checkout(driver, context):
    driver.find_element(By.ID, "checkout-btn").click()
    # Extract the newly generated Order ID from the UI
    order_id = driver.find_element(By.ID, "generated-order-id").text
    # SAVE the Order ID into our shared context dictionary!
    context['order_id'] = order_id
@then('the confirmation page should display the generated order number')
def verify_order_number(driver, context):
    # RETRIEVE the Order ID from the context
    expected_order_id = context['order_id']
    displayed_text = driver.find_element(By.ID, "confirmation-message").text
    assert expected_order_id in displayed_text
```

By passing the `context` fixture into any step definition, you can easily share dynamic data across your entire scenario!

---

## 2. Tags: Grouping and Filtering Scenarios

If you have 500 Gherkin scenarios, you rarely want to run all of them at once. Sometimes you just want to run a quick 5-minute `@smoke` test before a deployment.

Gherkin supports **Tags**, which perfectly integrate with Pytest's `-m` (marker) filtering!

**tests/features/payment.feature**

```gherkin
@regression @smoke @payments
Feature: Payment Gateway Processing
  @visa
  Scenario: Process valid Visa payment
    Given ...
  @mastercard
  Scenario: Process valid Mastercard payment
    Given ...
```

To execute these tags, simply pass the `-m` flag to your Pytest command:
- **Run all Smoke tests:** `pytest -m smoke`
- **Run ONLY the Visa test:** `pytest -m visa`
- **Run Smoke tests but EXCLUDE Payments:** `pytest -m "smoke and not payments"`

*Note: To prevent Pytest from throwing warnings about unknown markers, ensure you register your custom tags in your `pytest.ini` file!*

---

## 3. BDD Hooks: Automatically Capturing Failure Screenshots

Hooks are special functions that automatically execute at specific points during the test lifecycle. `pytest-bdd` provides several powerful hooks out of the box.

The most valuable hook is `pytest_bdd_step_error`. It executes the exact millisecond a step fails (due to an `AssertionError` or a missing element). We can use this hook to automatically tell Selenium to take a screenshot of the failure!

**tests/conftest.py**

```python
import pytest
from selenium import webdriver
@pytest.fixture(scope="function")
def driver(request):
    driver_instance = webdriver.Chrome()
    request.node.driver = driver_instance # Attach driver to request object
    yield driver_instance
    driver_instance.quit()
# The Hook: Executes only when a BDD step fails!
def pytest_bdd_step_error(request, feature, scenario, step, step_func, step_func_args, exception):
    print(f"\n[BDD ERROR] Scenario '{scenario.name}' failed at step: '{step.name}'")
    # Retrieve the driver instance we attached earlier
    driver = getattr(request.node, "driver", None)
    if driver:
        # Generate a unique filename and take a screenshot!
        screenshot_name = f"error_{scenario.name.replace(' ', '_')}.png"
        driver.save_screenshot(screenshot_name)
        print(f"[BDD ERROR] Screenshot saved as: {screenshot_name}")
```

Now, if a developer breaks the UI and our step `Then the confirmation page should display the generated order number` fails, Pytest will instantly snap a picture of the exact state of the browser, making debugging effortless!

## Conclusion

By mastering Context, Tags, and Hooks, you elevate `pytest-bdd` from a simple keyword mapper to an enterprise-grade automation framework.
- Use a **Context Dictionary Fixture** to seamlessly pass dynamic variables (like Order IDs) between isolated steps.
- Use **@tags** to selectively execute subsets of your massive feature suite.
- Use the **`pytest_bdd_step_error`** hook to automatically capture screenshots the moment a step fails!

In our next article, we will look at how to achieve massive Data-Driven BDD testing by iterating over Data Tables using **Scenario Outlines**!
