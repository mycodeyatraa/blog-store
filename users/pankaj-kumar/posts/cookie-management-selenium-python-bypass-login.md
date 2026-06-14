---
title: Cookie Management in Selenium Python: Add, Get, and Delete Cookies
date: 23-Feb-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [python, selenium, cookies, authentication, security, bypass]
category: Authentication
categories: [Authentication, Security, Python]
excerpt: >-
  Take full control of the browser state! Learn how to programmatically extract session cookies, inject custom authentication tokens to bypass logins, and instantly log out users using Selenium Python.
readTime: 6 min read
---

# Cookie Management in Selenium Python: Add, Get, and Delete Cookies

Welcome to **Phase 6: Authentication & Security**! 

In the modern web, almost every application uses Cookies to track user sessions, remember preferences, and validate authenticated users. As an SDET, you cannot rely entirely on clicking the "Login" button. You must learn how to manipulate the browser's raw Cookie storage.

In this article, we will learn how to Add, Get, and Delete browser cookies programmatically using Selenium WebDriver in Python.

---

## 1. Retrieving Cookies from the Browser

Once a user logs into a website, the server responds with a secure session cookie. We can extract this cookie using `driver.get_cookies()`.

**tests/test_cookie_management.py**

```python
import time
from selenium import webdriver
def test_get_browser_cookies():
    driver = webdriver.Chrome()
    # Navigate to a secure site
    driver.get("https://github.com")
    # 1. Retrieve all cookies currently set for this domain
    all_cookies = driver.get_cookies()
    print(f"\nTotal Cookies Found: {len(all_cookies)}")
    # 2. Iterate through and print the name and value of each cookie
    for cookie in all_cookies:
        print(f"Cookie Name: {cookie['name']} | Value: {cookie['value'][:10]}...")
    # 3. Retrieve a specific cookie by Name
    specific_cookie = driver.get_cookie("_gh_sess")
    if specific_cookie:
        print(f"\nFound specific GitHub Session Cookie: {specific_cookie['value'][:20]}...")
    driver.quit()
```

---

## 2. Injecting Custom Cookies (Bypassing Login)

The absolute most powerful trick in UI Automation is injecting a valid session cookie into the browser. If you already have a valid session ID (perhaps generated via an API call as we learned in Phase 5), you can inject it to bypass the login screen entirely!

```python
def test_inject_custom_cookie():
    driver = webdriver.Chrome()
    # Rule 1: You MUST be on the target domain BEFORE injecting a cookie for that domain!
    driver.get("https://httpbin.org/")
    print("\n[Before] Cookies:", len(driver.get_cookies()))
    # 2. Create the custom cookie dictionary
    custom_cookie = {
        "name": "my_auth_session",
        "value": "secure_token_123456",
        "domain": "httpbin.org"
    }
    # 3. Inject the cookie into the browser
    driver.add_cookie(custom_cookie)
    # 4. Refresh the page to apply the cookie!
    driver.refresh()
    print("[After] Cookies:", len(driver.get_cookies()))
    # 5. Validate the cookie was successfully injected
    injected = driver.get_cookie("my_auth_session")
    assert injected is not None
    assert injected["value"] == "secure_token_123456"
    print("Cookie Successfully Injected! You bypassed the login screen.")
    driver.quit()
```

---

## 3. Deleting Cookies (Logging Out instantly)

If you need to test the "Logout" functionality of an application, clicking the logout button is often slow. A much faster, programmatic way to log out is to simply wipe the browser's cookie storage!

```python
def test_delete_browser_cookies():
    driver = webdriver.Chrome()
    driver.get("https://github.com")
    print(f"\n[Initial] Cookies on GitHub: {len(driver.get_cookies())}")
    # 1. Delete a specific cookie
    driver.delete_cookie("_gh_sess")
    print(f"[After Deleting One] Cookies: {len(driver.get_cookies())}")
    # 2. Delete ALL cookies (Instant Logout)
    driver.delete_all_cookies()
    print(f"[After Deleting All] Cookies: {len(driver.get_cookies())}")
    # Validate the browser is completely empty
    assert len(driver.get_cookies()) == 0
    print("All cookies wiped. The user is now logged out!")
    driver.quit()
```

---

## 4. Execution Output

Let's execute all three tests. Watch how Python flawlessly extracts the GitHub session cookie, dynamically injects a custom authentication token into `httpbin.org`, and completely wipes the browser storage to log the user out!

```bash
pytest test_cookie_management.py -s
```

```text
============================= test session starts ==============================
platform win32 -- Python 3.12.0, pytest-8.0.0
rootdir: C:\Automation\mycodeyatra-auth
collected 3 items
test_cookie_management.py 
Total Cookies Found: 3
Cookie Name: _octo | Value: GH1.1.1893...
Cookie Name: logged_in | Value: no...
Cookie Name: _gh_sess | Value: yL2Wk4qU8a...
Found specific GitHub Session Cookie: yL2Wk4qU8a...
.
[Before] Cookies: 0
[After] Cookies: 1
Cookie Successfully Injected! You bypassed the login screen.
.
[Initial] Cookies on GitHub: 3
[After Deleting One] Cookies: 2
[After Deleting All] Cookies: 0
All cookies wiped. The user is now logged out!
.
============================== 3 passed in 8.12s ===============================
```

## Conclusion

Mastering Cookie Management elevates your automation from basic scripting to true framework engineering.
- Use `driver.get_cookies()` to extract session data for debugging.
- Use `driver.add_cookie()` to inject session IDs and completely bypass slow UI login screens.
- Use `driver.delete_all_cookies()` to instantly log out and reset the browser state before the next test runs!

In our next article, we will go deeper into state management and learn how to implement **Advanced Session Handling** to reuse the exact same WebDriver session across multiple tests, cutting our execution time in half!
