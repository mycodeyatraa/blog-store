---
title: Token Management in Selenium: Extracting JWTs from Local Storage
date: 09-Mar-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [python, selenium, jwt, local-storage, tokens, authentication]
category: Authentication
categories: [Authentication, Security, Python]
excerpt: >-
  Stop relying on cookies! Learn how to use Selenium's execute_script to extract and inject JWTs and API keys hidden inside the browser's Local Storage for modern React and Angular applications.
readTime: 5 min read
---

# Token Management in Selenium: Extracting JWTs from Local Storage

In traditional web applications, authentication relies heavily on HTTP Cookies. But with the rise of modern Single Page Applications (SPAs) built in React, Angular, or Vue, developers have shifted away from Cookies. 

Instead, when you log into a modern React application, the server responds with a JSON Web Token (JWT). The frontend JavaScript code takes that token and hides it inside the browser's **Local Storage** or **Session Storage**.

Because Selenium's `driver.get_cookies()` method *only* looks at Cookies, it is completely blind to Local Storage! In this article, we will learn how to execute custom JavaScript in Python to extract hidden API tokens.

---

## 1. Local Storage vs. Session Storage

- **Local Storage:** Persists even if you close the browser tab. Data is stored indefinitely until explicitly deleted.
- **Session Storage:** Data is wiped the moment the browser tab is closed.

Both storage engines are simply Key-Value dictionaries built into the browser. 

---

## 2. Using JavaScript Executor to Extract Tokens

To interact with Local Storage, we must use Selenium's `driver.execute_script()` method. This allows us to inject raw JavaScript into the browser engine and retrieve the results back into our Python variable.

**tests/test_token_extraction.py**

```python
from selenium import webdriver
def test_extract_jwt_from_local_storage():
    driver = webdriver.Chrome()
    driver.get("https://httpbin.org/")
    # 1. Simulate a React App saving a JWT into Local Storage
    print("\n[Browser] Injecting dummy JWT into Local Storage...")
    driver.execute_script("window.localStorage.setItem('auth_token', 'eyJhbGciOiJIUzI1Ni...SecureData');")
    # 2. Extract the JWT using Python!
    # We use 'return' in our JS string to send the data back to Python
    print("[Python] Attempting to extract token...")
    extracted_token = driver.execute_script("return window.localStorage.getItem('auth_token');")
    print(f"[Success] Extracted Token: {extracted_token}")
    assert extracted_token.startswith("eyJhb")
    # 3. Clear Local Storage to "Log Out"
    driver.execute_script("window.localStorage.removeItem('auth_token');")
    # or clear everything: driver.execute_script("window.localStorage.clear();")
    driver.quit()
```

---

## 3. Injecting Tokens to Bypass SPA Logins

Just like we bypassed logins using Cookie injection in Blog 44, we can bypass React logins using Local Storage injection!

If we already have a JWT (perhaps obtained via the `requests` library), we can inject it into the Local Storage before navigating to the secure dashboard.

```python
def test_inject_jwt_bypass():
    driver = webdriver.Chrome()
    # Rule 1: Navigate to the domain first
    driver.get("https://httpbin.org/")
    # 2. Inject the JWT into Local Storage
    my_api_token = "secure_token_abc123"
    driver.execute_script(f"window.localStorage.setItem('access_token', '{my_api_token}');")
    # 3. Navigate to the secure area (React will see the token and let us in!)
    print(f"\n[Bypass] Injected token '{my_api_token}' into Local Storage.")
    print("[Bypass] Navigating to Secure Dashboard...")
    driver.refresh()
    # Verify it exists
    verified = driver.execute_script("return window.localStorage.getItem('access_token');")
    assert verified == my_api_token
    print("[Success] Login Screen Bypassed!")
    driver.quit()
```

---

## 4. Execution Output

Let's execute both scripts. Watch how Python seamlessly bridges the gap between the browser's JavaScript V8 engine and our test automation framework!

```bash
pytest test_token_extraction.py -s
```

```text
============================= test session starts ==============================
platform win32 -- Python 3.12.0, pytest-8.0.0
rootdir: C:\Automation\mycodeyatra-auth
collected 2 items
test_token_extraction.py 
[Browser] Injecting dummy JWT into Local Storage...
[Python] Attempting to extract token...
[Success] Extracted Token: eyJhbGciOiJIUzI1Ni...SecureData
.
[Bypass] Injected token 'secure_token_abc123' into Local Storage.
[Bypass] Navigating to Secure Dashboard...
[Success] Login Screen Bypassed!
.
============================== 2 passed in 3.42s ===============================
```

## Conclusion

Modern web architecture requires modern testing techniques.
- Selenium's `driver.get_cookies()` will not find JWTs stored in React/Angular applications.
- You must use `driver.execute_script("return window.localStorage.getItem('key');")` to extract hidden tokens.
- You can instantly log a user out of a React app by executing `window.localStorage.clear();`.
- Combine this technique with the Python `requests` library to fetch JWTs via backend APIs and inject them into the browser for ultimate UI bypasses!

This officially concludes **Phase 6: Authentication & Security!** You are now an expert in handling Cookies, SSO, MFA, Session hijacking, and Local Storage.
