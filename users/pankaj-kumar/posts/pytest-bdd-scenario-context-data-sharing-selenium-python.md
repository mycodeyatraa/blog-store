---
title: Sharing Data: Managing Scenario Context in pytest-bdd
date: 10-Mar-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [python, selenium, bdd, pytest-bdd, scenario-context, state-management, fixtures]
category: Selenium Python
categories: [Selenium Python, Behavior Driven Development]
excerpt: >-
  When a Given step creates an Order ID, how does the Then step verify it? Learn how to share state between isolated step definitions using Pytest fixtures and dataclasses without relying on global variables.
readTime: 4 min read
---

# Sharing Data: Managing Scenario Context in pytest-bdd

In a typical procedural Python test, if you want to share data between two lines of code, you just assign a variable. 

```python
def test_checkout():
    order_id = checkout_page.submit_order()
    assert order_page.verify_order_exists(order_id)
```

However, Behavior Driven Development completely shatters this model. Because every single `Given`, `When`, and `Then` is a completely isolated Python function, they do not inherently share a local variable scope!

If a `@when` step clicks "Checkout" and the website generates a dynamic `Order ID #12345`, how does the `@then` step know what order ID to assert? 

In this tutorial, we will learn how to elegantly share state between isolated step definitions using **Scenario Context**.

---

## 1. The Anti-Pattern: Global Variables

The instinct of many junior automation engineers is to simply declare a global variable at the top of the file:

```python
global_order_id = None
@when("I submit the order")
def submit_order(driver):
    global global_order_id
    global_order_id = driver.find_element("id", "order-id").text
@then("the order ID should be visible on the dashboard")
def verify_order(driver):
    assert global_order_id in driver.page_source
```

**Why is this terrible?**
1. It breaks parallel execution! If two scenarios run simultaneously, they will overwrite each other's global variable.
2. The state bleeds between scenarios. If Scenario 1 sets the variable and Scenario 2 forgets to clear it, Scenario 2 might pass using Scenario 1's data!

---

## 2. The Pytest Way: Dictionary Fixtures

Because `pytest-bdd` is built entirely on top of `pytest`, the absolute best way to share context within a single scenario is by using a **Pytest Fixture** that yields a dictionary. 

Fixtures defined with `scope="function"` are destroyed and recreated for every single scenario, completely eliminating cross-contamination.

Let's implement our Scenario Context dictionary in `conftest.py`:

```python
import pytest
@pytest.fixture(scope="function")
def context():
    """
    A dictionary used to share state between steps in a single scenario.
    It is completely wiped clean when the scenario ends.
    """
    return {}
```

---

## 3. Passing Data Between Steps

Now that we have our `context` fixture, we can simply inject it into any step definition as an argument, just like we do with the `driver` fixture!

Let's look at a complete checkout flow:

```gherkin
Feature: E-Commerce Checkout
  Scenario: Purchasing an item
    Given I navigate to the store
    When I submit a new order
    Then the exact order ID should appear in my history
```

And here is the Python implementation:

```python
from pytest_bdd import given, when, then
@given("I navigate to the store")
def navigate_to_store(driver):
    driver.get("https://practice.mycodeyatra.com/store")
@when("I submit a new order")
def submit_order(driver, context):
    # Click checkout
    driver.find_element("id", "checkout-btn").click()
    # Extract the dynamically generated order ID from the UI
    dynamic_id = driver.find_element("id", "generated-order-id").text
    # SAVE it to the scenario context dictionary!
    context["order_id"] = dynamic_id
    print(f"Captured Order ID: {dynamic_id}")
@then("the exact order ID should appear in my history")
def verify_order_history(driver, context):
    driver.get("https://practice.mycodeyatra.com/history")
    # RETRIEVE the saved order ID from the context!
    expected_id = context.get("order_id")
    assert expected_id is not None, "Order ID was never captured in the previous step!"
    history_text = driver.find_element("id", "history-table").text
    assert expected_id in history_text, f"Order {expected_id} was missing from history!"
```

---

## 4. Best Practices for Scenario Context

While the dictionary approach is incredibly powerful, it can get messy if developers are using different keys for the same data (e.g., one person uses `context["order_id"]` and another uses `context["OrderID"]`).

### The Data Class Pattern
For massive enterprise frameworks, you can upgrade your dictionary fixture to a strictly typed Data Class or Pydantic model:

```python
import pytest
from dataclasses import dataclass
@dataclass
class ScenarioContext:
    order_id: str = None
    user_email: str = None
    total_price: float = 0.0
@pytest.fixture(scope="function")
def context():
    return ScenarioContext()
```

Now, when you inject `context` into your steps, your IDE will provide auto-complete for `context.order_id`!

## Conclusion

By leveraging a `scope="function"` fixture, we have created an isolated, thread-safe memory bank for every single BDD scenario. This allows us to capture dynamic data in a `@when` step and gracefully assert it in a `@then` step without resorting to dangerous global variables.

In the next tutorial, we will explore **Data Tables**, a powerful Gherkin syntax that allows us to pass massive lists of data directly from our `.feature` files into our Python step definitions!
