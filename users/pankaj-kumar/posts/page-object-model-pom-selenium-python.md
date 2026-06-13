---
title: The Page Object Model (POM) in Selenium Python
date: 29-Dec-2024
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [selenium, python, page-object-model, pom, framework, architecture]
category: Framework Architecture
categories: [Framework Architecture, Selenium, Python]
excerpt: >-
  Build invincible test frameworks! Learn how to implement the Page Object Model (POM) in Selenium Python to separate locators from logic and eliminate maintenance nightmares.
readTime: 6 min read
---

# The Page Object Model (POM) in Selenium Python

If you have been following this series, your test scripts likely look something like this:

```python
def test_login_success():
    driver = webdriver.Chrome()
    driver.get("https://mycodeyatra.com/login")
    driver.find_element(By.ID, "username").send_keys("admin")
    driver.find_element(By.ID, "password").send_keys("password123")
    driver.find_element(By.ID, "login-btn").click()
    assert "Dashboard" in driver.title
```

This works perfectly... until the frontend developer changes the ID of the username field from `username` to `email_input`. 

If you have 50 tests that log in, you now have to perform a "Find & Replace" across 50 different Python files! This makes your framework incredibly fragile and impossible to maintain at an enterprise scale.

The solution is the **Page Object Model (POM)**.

---

## 1. What is the Page Object Model?

The Page Object Model is an object-oriented design pattern. The core rule is simple: **Test files should never contain Selenium Locators (`By.ID`).**

Instead, for every physical web page in your application (e.g., Login Page, Dashboard Page, Profile Page), you create a corresponding Python Class. 
- The **Variables** of the class store the locators for that page.
- The **Methods** of the class represent the actions a user can take on that page.

Let's refactor our brittle test into a robust POM architecture.

---

## 2. Creating the Page Class

First, we create a file named `login_page.py`. This file will *only* contain locators and actions. It will **not** contain any `pytest` assertions or test data.

**pages/login_page.py**

```python
from selenium.webdriver.common.by import By
class LoginPage:
    # 1. Constructor: Receive the driver from the test
    def __init__(self, driver):
        self.driver = driver
    # 2. Locators: Stored as Class-level tuples
    URL = "https://mycodeyatra.com/login"
    USERNAME_INPUT = (By.ID, "username")
    PASSWORD_INPUT = (By.ID, "password")
    LOGIN_BUTTON = (By.ID, "login-btn")
    # 3. Actions: Methods that interact with the locators
    def load(self):
        self.driver.get(self.URL)
    def enter_username(self, username):
        self.driver.find_element(*self.USERNAME_INPUT).send_keys(username)
    def enter_password(self, password):
        self.driver.find_element(*self.PASSWORD_INPUT).send_keys(password)
    def click_login(self):
        self.driver.find_element(*self.LOGIN_BUTTON).click()
    # 4. High-Level Action: Combines smaller actions into a single reusable flow
    def login(self, username, password):
        self.load()
        self.enter_username(username)
        self.enter_password(password)
        self.click_login()
```

*(Notice the `*` before `self.USERNAME_INPUT`. Because our locator is a tuple `(By.ID, "username")`, we use the asterisk to unpack the tuple into two separate arguments for the `find_element` method!)*

---

## 3. Writing the POM Test

Now, let's write the actual PyTest file (`test_login.py`). 

Notice how clean and readable it becomes! It reads like plain English.

**tests/test_login.py**

```python
import pytest
from selenium import webdriver
from pages.login_page import LoginPage
@pytest.fixture
def driver():
    driver = webdriver.Chrome()
    yield driver
    driver.quit()
def test_successful_login(driver):
    # 1. Initialize the Page Object
    login_page = LoginPage(driver)
    # 2. Perform the high-level action
    login_page.login("admin", "password123")
    # 3. Assert the result (Assertions stay in the test file!)
    assert "Dashboard" in driver.title
```

### Why is this better?
If the frontend developer changes the username ID to `email_input` tomorrow, you only have to update **one single line** of code in `login_page.py`. 
You do not have to touch any of the 50 test files that use the Login Page!

---

## 4. Returning Page Objects (Fluent POM)

When you click "Login", the browser transitions to the Dashboard page. 

In a highly advanced POM framework, actions that cause a page transition should automatically return the Page Object of the *next* page! This creates a "Fluent Interface".

**pages/login_page.py (Updated)**

```python
from pages.dashboard_page import DashboardPage
class LoginPage:
    # ... previous code ...
    def click_login(self):
        self.driver.find_element(*self.LOGIN_BUTTON).click()
        # Return the next page!
        return DashboardPage(self.driver)
```

**tests/test_fluent.py**

```python
def test_fluent_login(driver):
    login_page = LoginPage(driver)
    login_page.load()
    login_page.enter_username("admin")
    login_page.enter_password("pass")
    # click_login() returns a DashboardPage object!
    dashboard_page = login_page.click_login()
    # We instantly have access to dashboard methods without initializing it!
    dashboard_page.click_logout()
```

---

## 5. Execution Output

When we execute our POM-structured tests in PyTest, the terminal output remains exactly the same, but our underlying architecture is now enterprise-grade!

```text
============================= test session starts ==============================
platform win32 -- Python 3.12.0, pytest-8.0.0
rootdir: C:\Automation\mycodeyatra-tests
collected 2 items
test_login.py::test_successful_login PASSED                              [ 50%]
test_fluent.py::test_fluent_login PASSED                                 [100%]
============================== 2 passed in 10.15s ==============================
```

## Conclusion

The Page Object Model (POM) is the absolute gold standard for UI automation.
- **Pages** contain Locators and Actions.
- **Tests** contain Data and Assertions.

Never mix them! By strictly separating the "How" (finding elements) from the "What" (verifying behavior), your test suite becomes invincible to UI changes.

In our next article, we will solve another architectural issue: Hardcoded `webdriver.Chrome()` instances scattered across our files. We will introduce the **Driver Factory Pattern**!
