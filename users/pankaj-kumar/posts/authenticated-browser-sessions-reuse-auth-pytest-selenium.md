---
title: Authenticated Browser Sessions: Reusing State Across Pytest Suites
date: 11-Mar-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [python, selenium, pytest, authentication, cookies, performance]
category: Authentication
categories: [Authentication, Security, Python]
excerpt: >-
  Supercharge your test execution time! Learn how to extract authentication cookies once, save them to a JSON file, and instantly inject them into every test using Pytest fixtures to completely bypass UI logins.
readTime: 6 min read
---

# Authenticated Browser Sessions: Reusing State Across Pytest Suites

Throughout Phase 6, we have learned the building blocks of security: Cookies, Tokens, Single Sign-On, and Local Storage.

However, if you have a suite of 500 tests, you absolutely do not want to execute the slow, complex login flow 500 times! It will waste hours of test execution time.

In this final article on Authentication, we will use Python's `pytest` framework to create a massive performance optimization: **Global Session Authentication**. We will log in exactly *once* at the start of the test suite, save the authentication state to a JSON file, and then instantly inject that state into every subsequent test!

---

## 1. Saving the Authentication State

The strategy is simple. We will write a script that opens the browser, navigates the UI login screen, and extracts the cookies. Instead of injecting them immediately, we will write those cookies to a local `auth_state.json` file on our hard drive.

**utils/auth_manager.py**

```python
import json
from selenium import webdriver
from selenium.webdriver.common.by import By
def generate_global_auth_state():
    driver = webdriver.Chrome()
    driver.get("https://github.com/login")
    # 1. Execute the slow UI Login exactly once
    print("\n[Auth] Executing one-time UI login...")
    driver.find_element(By.ID, "login_field").send_keys("test_automation_user")
    driver.find_element(By.ID, "password").send_keys("SecurePassword123!")
    driver.find_element(By.NAME, "commit").click()
    # 2. Extract the authenticated cookies
    cookies = driver.get_cookies()
    # 3. Save the cookies to a JSON file on the hard drive
    with open("auth_state.json", "w") as file:
        json.dump(cookies, file)
    print(f"[Auth] Saved {len(cookies)} cookies to auth_state.json!")
    driver.quit()
if __name__ == "__main__":
    generate_global_auth_state()
```

---

## 2. Injecting State via Pytest Fixtures

Now that our cookies are saved to disk, we can use a `pytest` fixture to automatically load and inject them before every test runs. 

By setting the fixture scope to `function`, every test gets a fresh browser window, but that window is instantly injected with our pre-authenticated cookies!

**tests/test_authenticated_suite.py**

```python
import json
import pytest
from selenium import webdriver
# Pytest fixture that runs before every test
@pytest.fixture
def authenticated_driver():
    driver = webdriver.Chrome()
    # 1. Navigate to the domain (Required before injecting cookies!)
    driver.get("https://github.com")
    # 2. Load the cookies from our JSON file
    with open("auth_state.json", "r") as file:
        saved_cookies = json.load(file)
    # 3. Inject the cookies instantly
    for cookie in saved_cookies:
        driver.add_cookie(cookie)
    # 4. Refresh to apply the authentication state!
    driver.refresh()
    # Yield the fully logged-in driver to the test
    yield driver
    # Teardown
    driver.quit()
def test_dashboard_access(authenticated_driver):
    # This test starts ALREADY logged in!
    authenticated_driver.get("https://github.com/dashboard")
    print("\n[Test 1] Instantly accessed secure dashboard without logging in!")
    assert "dashboard" in authenticated_driver.current_url
def test_profile_access(authenticated_driver):
    # This test also starts ALREADY logged in!
    authenticated_driver.get("https://github.com/settings/profile")
    print("\n[Test 2] Instantly accessed secure profile without logging in!")
    assert "settings" in authenticated_driver.current_url
```

---

## 3. Execution Output

Let's execute the suite. We first run `auth_manager.py` to generate the token, and then we run `pytest` to execute our tests. Watch how both tests bypass the login screen entirely and instantly access secure pages!

```bash
python utils/auth_manager.py
```
```text
[Auth] Executing one-time UI login...
[Auth] Saved 5 cookies to auth_state.json!
```

```bash
pytest test_authenticated_suite.py -s
```
```text
============================= test session starts ==============================
platform win32 -- Python 3.12.0, pytest-8.0.0
rootdir: C:\Automation\mycodeyatra-auth
collected 2 items
test_authenticated_suite.py 
[Test 1] Instantly accessed secure dashboard without logging in!
.
[Test 2] Instantly accessed secure profile without logging in!
.
============================== 2 passed in 3.15s ===============================
```

Both tests finished in just 3 seconds because neither of them had to wait 10 seconds for the UI login screen!

## Conclusion

Reusing authentication state is the #1 optimization secret of Enterprise SDETs.
- Write a setup script that logs in once and saves the `driver.get_cookies()` payload to a `.json` file.
- Use a `@pytest.fixture` to read that file and `driver.add_cookie()` into the browser.
- Your entire suite of 500 tests will now run independently, in parallel, without ever interacting with a login screen!

You have officially conquered **Phase 6: Authentication & Security!**

In the next section of our curriculum, **Phase 7: Security Testing**, we will pivot into true DevSecOps. We will learn how to validate HTTP Security Headers, parse Content Security Policies (CSP), and integrate OWASP ZAP to execute automated penetration testing right alongside your Selenium scripts!
