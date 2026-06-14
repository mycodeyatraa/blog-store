---
title: pytest-bdd Basics: Mapping Feature Files to Step Definitions
date: 15-May-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [python, selenium, bdd, pytest-bdd, gherkin]
category: Behavior Driven Development
categories: [Behavior Driven Development, Python, Automation]
excerpt: >-
  Learn how to execute Gherkin feature files in Python! We dive deep into pytest-bdd, mapping Given/When/Then scenarios to WebDriver Step Definitions using parsers and fixtures.
readTime: 6 min read
---

# pytest-bdd Basics: Mapping Feature Files to Step Definitions

In our previous article, we learned how to write beautiful, human-readable test scenarios using Gherkin syntax. But how do we actually make those text files control a browser? 

In the Python ecosystem, the most popular BDD framework is **`pytest-bdd`**. It integrates directly with the standard Pytest runner, allowing you to use all your existing Pytest fixtures and plugins (like `pytest-xdist` and `allure-pytest`) alongside your BDD tests!

In this article, we will teach you how to write a `.feature` file and map it to Python code using **Step Definitions**.

---

## 1. Installing pytest-bdd

First, install the library into your Python environment:

```bash
pip install pytest-bdd
```

Next, create a new directory structure for your BDD tests. It is standard practice to separate your Gherkin `.feature` files from your Python `.py` step definition files.

```text
tests/
├── features/
│   └── login.feature
└── step_defs/
    ├── conftest.py
    └── test_login_steps.py
```

---

## 2. Writing the Feature File

Create the Gherkin file. Notice we are adding the `@login` tag. Tags allow us to filter which tests we want to run later from the command line!

**tests/features/login.feature**

```gherkin
@login
Feature: User Authentication
  Scenario: Successful login with valid credentials
    Given the user navigates to the login page
    When the user enters the username "admin"
    And the user enters the password "SecurePass123"
    And the user clicks the login button
    Then the user should be redirected to the dashboard
```

---

## 3. Writing the Step Definitions

Now, we must write Python code to match every single step in our feature file.

In `pytest-bdd`, step definitions are standard Python functions decorated with `@given`, `@when`, or `@then`. The string inside the decorator must *exactly* match the text in the Gherkin file.

**tests/step_defs/test_login_steps.py**

```python
from pytest_bdd import scenarios, given, when, then, parsers
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
# 1. Load the feature file!
scenarios('../features/login.feature')
# 2. Step Definitions
@given('the user navigates to the login page')
def navigate_to_login(driver):
    driver.get("https://mycodeyatra.com/login")
# We use "parsers.parse" to capture the string "admin" from the Gherkin step!
@when(parsers.parse('the user enters the username "{username}"'))
def enter_username(driver, username):
    driver.find_element(By.ID, "username").send_keys(username)
@when(parsers.parse('the user enters the password "{password}"'))
def enter_password(driver, password):
    driver.find_element(By.ID, "password").send_keys(password)
@when('the user clicks the login button')
def click_login(driver):
    driver.find_element(By.ID, "login-btn").click()
@then('the user should be redirected to the dashboard')
def verify_dashboard_redirect(driver):
    # Wait for the URL to change to the dashboard
    WebDriverWait(driver, 10).until(
        EC.url_contains("/dashboard")
    )
    assert "dashboard" in driver.current_url.lower()
```

### How does this work?
1. The `scenarios()` function tells Pytest to scan the `login.feature` file and generate a test case for every `Scenario` it finds.
2. When Pytest executes the test, it reads the Gherkin steps line by line.
3. For each line, it searches this Python file for a decorator that matches the text.
4. If it finds a match, it executes the Python function!
5. Notice we used `parsers.parse()` in the `@when` decorator. This allows us to capture the dynamic text `"admin"` from the feature file and pass it as an argument (`username`) directly into the Python function!

---

## 4. Reusing the Pytest WebDriver Fixture

You might be wondering: "Where did the `driver` argument come from in those step definition functions?"

Because `pytest-bdd` is fully integrated with Pytest, it automatically inherits all the fixtures from your `conftest.py` file! 

**tests/step_defs/conftest.py**

```python
import pytest
from selenium import webdriver
@pytest.fixture(scope="function")
def driver():
    # Initialize Chrome
    driver_instance = webdriver.Chrome()
    driver_instance.maximize_window()
    # Pass the driver to the BDD steps
    yield driver_instance
    # Teardown
    driver_instance.quit()
```
Whenever a step definition requests `driver` as a parameter, Pytest automatically injects the WebDriver instance from `conftest.py`. This ensures we are using the exact same browser window across all of our Given/When/Then steps!

---

## 5. Execution Output

To run the BDD suite, you just use the standard `pytest` command. Since we tagged our scenario with `@login`, we can use the `-m` (marker) flag to execute only that specific feature!

```bash
pytest tests/step_defs/ -m login -v
```

```text
============================= test session starts ==============================
collected 1 item
tests/step_defs/test_login_steps.py::test_successful_login_with_valid_credentials PASSED
============================== 1 passed in 4.12s ===============================
```

*Note: Pytest automatically generates the test name `test_successful_login_with_valid_credentials` based on the name of the Gherkin Scenario!*

## Conclusion

With `pytest-bdd`, bridging the gap between business logic and automation code is effortless.
- Use `scenarios()` to link a Gherkin `.feature` file to a Python `.py` file.
- Use `@given`, `@when`, and `@then` decorators to map plain English steps to Python functions.
- Use `parsers.parse()` to extract dynamic string variables (like usernames and passwords) from the Gherkin text.
- Rely on your existing `conftest.py` fixtures to manage your WebDriver instance!

In our next article, we will explore advanced `pytest-bdd` concepts, including **Scenario Context** (how to pass data between steps) and **Hooks** (how to execute code before and after specific BDD steps)!
