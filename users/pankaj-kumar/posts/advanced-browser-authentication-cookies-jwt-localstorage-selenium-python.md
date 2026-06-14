---
title: Advanced Browser Authentication: Cookies, JWTs, and LocalStorage
date: 12-Jul-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [python, selenium, authentication, cookies, jwt, localstorage]
category: Authentication & Security
categories: [Authentication & Security, Python, Automation]
excerpt: >-
  Stop wasting 20 minutes logging in through the UI! Learn how to use Python requests to authenticate via API and inject Session Cookies or JWTs directly into the browser's LocalStorage to bypass the login screen.
readTime: 6 min read
---

# Advanced Browser Authentication: Cookies, JWTs, and LocalStorage

If your automation suite has 500 test cases, and each test case starts by typing a username, typing a password, and clicking "Login", your suite is incredibly inefficient. 

UI authentication is slow. It takes ~3 seconds to load the login page and interact with the elements. Over 500 tests, that's **25 minutes** wasted just logging in! Furthermore, repeatedly slamming your company's SSO (Single Sign-On) server might trigger rate-limiting or anti-bot protections.

In this article, we will teach you how to **Bypass the UI Login Screen**. We will log in once via a fast backend API, steal the resulting authentication token, and inject it directly into the browser's Cookies or LocalStorage!

---

## 1. Injecting Authentication Cookies

Many traditional web applications (like Django, PHP, or older Spring Boot apps) manage sessions using HTTP Cookies (usually named `session_id` or `JSESSIONID`).

If we know the valid `session_id`, we can add it to our Selenium browser and skip the login page entirely.

**How to Inject a Cookie:**
1. You must navigate to the target domain *first* (Selenium cannot set a cookie for `mycodeyatra.com` if the browser is sitting on `google.com`).
2. Use `driver.add_cookie()`.
3. Refresh the page to apply the authenticated state.

**tests/test_cookie_auth.py**

```python
import pytest
import requests
from selenium import webdriver
def get_session_cookie_via_api():
    """Logs in via API and returns the session cookie value in 100ms."""
    payload = {"username": "admin", "password": "SuperSecretPassword!"}
    response = requests.post("https://api.mycodeyatra.com/v1/auth", json=payload)
    return response.cookies.get("session_id")
def test_dashboard_with_cookie_injection():
    # 1. Get the token instantly via API
    session_token = get_session_cookie_via_api()
    driver = webdriver.Chrome()
    # 2. Navigate to the domain (We just hit the homepage, NOT the login page)
    driver.get("https://mycodeyatra.com")
    # 3. Inject the Cookie
    cookie_dict = {
        'name': 'session_id',
        'value': session_token,
        'domain': 'mycodeyatra.com',
        'path': '/',
        'secure': True
    }
    driver.add_cookie(cookie_dict)
    # 4. Navigate directly to the protected Dashboard!
    driver.get("https://mycodeyatra.com/dashboard")
    # Assert we are logged in without ever seeing the Login UI
    assert "Welcome back, Admin" in driver.page_source
    driver.quit()
```

---

## 2. Injecting JWTs into LocalStorage

Modern Single Page Applications (React, Angular, Vue) usually do not use Cookies for authentication. Instead, they use **JSON Web Tokens (JWTs)**. When you log in, the server returns a massive encoded string, and the front-end JavaScript saves it inside the browser's `LocalStorage` or `SessionStorage`.

Selenium does not have a native `driver.add_local_storage()` method. To manipulate LocalStorage, we must execute raw JavaScript using `driver.execute_script()`!

**tests/test_jwt_auth.py**

```python
import pytest
import requests
from selenium import webdriver
def get_jwt_via_api():
    """Logs in via API and returns the JWT Bearer token."""
    payload = {"username": "admin", "password": "SuperSecretPassword!"}
    response = requests.post("https://api.mycodeyatra.com/v1/login", json=payload)
    return response.json().get("access_token")
def test_dashboard_with_jwt_injection():
    # 1. Fetch the JWT via API
    jwt_token = get_jwt_via_api()
    driver = webdriver.Chrome()
    # 2. Navigate to the domain so LocalStorage is initialized for that origin
    driver.get("https://mycodeyatra.com")
    # 3. Inject the JWT into LocalStorage using JavaScript!
    # Note: You must know the EXACT key your React app expects (e.g., 'authToken')
    js_command = f"window.localStorage.setItem('authToken', '{jwt_token}');"
    driver.execute_script(js_command)
    # 4. Refresh or Navigate to the protected route
    driver.get("https://mycodeyatra.com/dashboard")
    # The React app will read the token from LocalStorage and grant access instantly!
    assert "Secure Data" in driver.page_source
    driver.quit()
```

---

## 3. Creating a Pytest Fixture for Global Authentication

If you want all 500 of your tests to run as the Admin user, you should move this injection logic into an `autouse=True` Pytest fixture in your `conftest.py`.

**conftest.py**

```python
import pytest
import requests
from selenium import webdriver
@pytest.fixture(scope="session")
def admin_jwt():
    """Fetch the token ONCE per test suite run."""
    payload = {"username": "admin", "password": "SuperSecretPassword!"}
    response = requests.post("https://api.mycodeyatra.com/v1/login", json=payload)
    return response.json().get("access_token")
@pytest.fixture(scope="function")
def authenticated_driver(admin_jwt):
    """Provides a WebDriver instance that is pre-authenticated."""
    driver = webdriver.Chrome()
    # Navigate and Inject
    driver.get("https://mycodeyatra.com")
    driver.execute_script(f"window.localStorage.setItem('authToken', '{admin_jwt}');")
    yield driver
    driver.quit()
```

Now, any test function that requests the `authenticated_driver` fixture will instantly open a browser that is already logged in as an Admin!

## Conclusion

UI Automation should only test the UI. If you are testing the "Add to Cart" functionality, you shouldn't be wasting time and resources testing the "Login" functionality at the start of that test.
- Use `requests` to log in via the backend API in milliseconds.
- If the app uses Cookies, use `driver.add_cookie()`.
- If the app uses JWTs, use `driver.execute_script("window.localStorage.setItem(...)")`.
- Wrap this logic in a Pytest Fixture to instantly authenticate every browser session across your entire suite!

In our next article, we will look at the final missing topic: **Accessibility Testing** using Axe-Core!
