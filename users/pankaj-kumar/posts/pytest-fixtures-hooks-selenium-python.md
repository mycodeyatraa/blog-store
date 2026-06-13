---
title: Mastering PyTest Fixtures and Hooks for Selenium Python
date: 10-Jan-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [selenium, python, pytest, fixtures, hooks, screenshots]
category: Framework Architecture
categories: [Framework Architecture, Selenium, Python]
excerpt: >-
  Take control of your test lifecycle! Learn how to use PyTest Fixtures for setup/teardown and PyTest Hooks to automatically capture Selenium screenshots on failure.
readTime: 7 min read
---

# Mastering PyTest Fixtures and Hooks for Selenium Python

Throughout this entire series, we have been using `@pytest.fixture` to initialize our Selenium WebDrivers. But what exactly *is* a fixture? And what are PyTest "Hooks"?

To build an enterprise-grade automation framework, you must understand how PyTest manages the lifecycle of your tests. You need to know how to set up preconditions (like logging into a database), share context between tests, and automatically capture screenshots if a test fails.

In this final article of Phase 3, we will master PyTest Fixtures and Hooks!

---

## 1. What is a Fixture? (Setup & Teardown)

In traditional frameworks (like JUnit or TestNG), you use functions named `setUp()` and `tearDown()` that run before and after every single test.

PyTest uses **Fixtures**. A fixture is a Python function that uses the `yield` keyword to split the setup from the teardown.

```python
import pytest
from selenium import webdriver
@pytest.fixture
def driver():
    # --- SETUP PHASE ---
    print("\n[Setup] Initializing the Chrome browser...")
    driver_instance = webdriver.Chrome()
    # Pause the fixture, pass the driver to the test, and wait for the test to finish!
    yield driver_instance 
    # --- TEARDOWN PHASE ---
    print("\n[Teardown] Closing the browser...")
    driver_instance.quit()
def test_google_title(driver):
    print("Executing the actual test logic...")
    driver.get("https://google.com")
    assert "Google" in driver.title
```

When you run this test, PyTest automatically calls the setup code, hands the `driver` to `test_google_title()`, waits for the test to pass (or fail), and then perfectly executes the teardown code!

---

## 2. Fixture Scope: Function vs Session

By default, PyTest executes the setup and teardown for *every single test function*. If you have 100 tests, PyTest will open and close Chrome 100 times.

Sometimes, opening a browser is too slow. What if you want to open the browser *once*, run all 100 tests inside the same browser window, and then close it? 

You do this by changing the **Scope** of the fixture from `function` to `session`.

```python
# This driver will only be created ONCE per test run!
@pytest.fixture(scope="session")
def session_driver():
    driver = webdriver.Chrome()
    yield driver
    driver.quit()
def test_login(session_driver):
    session_driver.get("https://mycodeyatra.com/login")
    assert "Login" in session_driver.title
def test_dashboard(session_driver):
    # This uses the EXACT SAME browser window from test_login!
    session_driver.get("https://mycodeyatra.com/dashboard")
    assert "Dashboard" in session_driver.title
```

*Note: Session-scoped drivers break Test Isolation. If `test_login` crashes the browser, `test_dashboard` will also fail! Use `session` scope sparingly, mostly for database connections or API tokens.*

---

## 3. PyTest Hooks: Tapping into the Engine

Fixtures handle *data* and *objects*. But what if you want to change how PyTest actually behaves?

What if you want to automatically take a Selenium screenshot every time a test fails, without writing a `try/except` block in every single test?

We do this using **PyTest Hooks**. Hooks are special, reserved function names that you place inside your `conftest.py` file. PyTest calls these functions internally during its execution loop.

We will use the `pytest_runtest_makereport` hook to intercept test failures!

**conftest.py**

```python
import pytest
from selenium import webdriver
# We need a way to track the driver globally so the Hook can access it!
driver_instance = None
@pytest.fixture
def driver():
    global driver_instance
    driver_instance = webdriver.Chrome()
    yield driver_instance
    driver_instance.quit()
# This Hook is called by PyTest automatically after every setup, call, and teardown!
@pytest.hookimpl(tryfirst=True, hookwrapper=True)
def pytest_runtest_makereport(item, call):
    # Execute all other hooks to obtain the report object
    outcome = yield
    rep = outcome.get_result()
    # We only care about the actual test 'call' (not setup/teardown), and only if it failed!
    if rep.when == 'call' and rep.failed:
        # Check if the global driver is alive
        if driver_instance is not None:
            # Dynamically grab the test name to name the screenshot
            test_name = item.name
            screenshot_path = f"{test_name}_failure.png"
            # Take the screenshot!
            driver_instance.save_screenshot(screenshot_path)
            print(f"\n[FAILURE HOOK] Captured screenshot: {screenshot_path}")
```

Now, write a test that intentionally fails:

**test_hooks.py**

```python
def test_failing_logic(driver):
    driver.get("https://mycodeyatra.com")
    assert "Wrong Title" in driver.title
```

---

## 4. Execution Output

When we run our test, the assertion fails as expected. But look at the output! Our PyTest Hook intercepted the failure instantly and triggered a Selenium screenshot before the driver was torn down.

```text
============================= test session starts ==============================
platform win32 -- Python 3.12.0, pytest-8.0.0
rootdir: C:\Automation\mycodeyatra-tests
collected 1 item
test_hooks.py::test_failing_logic FAILED                                 [100%]
=================================== FAILURES ===================================
______________________________ test_failing_logic ______________________________
driver = <selenium.webdriver.chrome.webdriver.WebDriver ...>
    def test_failing_logic(driver):
        driver.get("https://mycodeyatra.com")
>       assert "Wrong Title" in driver.title
E       AssertionError: assert 'Wrong Title' in 'MyCodeYatra'
test_hooks.py:3: AssertionError
----------------------------- Captured stdout call -----------------------------
[FAILURE HOOK] Captured screenshot: test_failing_logic_failure.png
=========================== 1 failed in 5.12s ==================================
```

## Conclusion & End of Phase 3

Hooks and Fixtures are the beating heart of a Python automation framework.
- Use **Fixtures** to manage `webdriver` instances, API tokens, and database connections. Use scopes (`function`, `class`, `module`, `session`) to control performance vs isolation.
- Use **Hooks** in `conftest.py` to modify PyTest's behavior, add command-line arguments, and integrate automated reporting like failure screenshots!

Congratulations! You have officially completed **Phase 3: Test Design & Architecture**. 

We have learned how to use POM, Builders, Data-Driven strategies, and PyTest hooks. However, our framework is currently just a bunch of scattered Python files.

In **Phase 4: Framework**, we will tie everything together by building a professional, enterprise-grade folder structure complete with Configuration Managers, Utility Classes, and Python Logging. See you there!
