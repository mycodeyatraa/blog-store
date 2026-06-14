---
title: Validating Security Headers in Python: A DevSecOps Guide
date: 15-Mar-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [python, selenium, security, devsecops, headers, cdp]
category: Security
categories: [Security, Python, DevSecOps]
excerpt: >-
  Protect your application from MitM and Clickjacking! Learn how to write automated tests in Python to validate critical HTTP Security Headers like HSTS and X-Frame-Options using both Requests and Selenium CDP.
readTime: 6 min read
---

# Validating Security Headers in Python: A DevSecOps Guide

If you are an SDET, your job shouldn't stop at verifying if the "Login" button works. Enterprise QA engineers are expected to validate **Security**. One of the most critical aspects of application security is HTTP Security Headers.

If your application does not enforce `Strict-Transport-Security` or `X-Frame-Options`, it is highly vulnerable to Man-in-the-Middle (MitM) attacks and Clickjacking. 

In this article, we will teach you how to write automated tests that intercept network traffic and validate that your DevOps team has configured the server's security headers correctly!

---

## 1. What are Security Headers?

When a web server responds to a browser, it sends metadata called HTTP Headers. **Security Headers** are specific instructions telling the browser how to behave defensively. 

The top 4 critical Security Headers every application must have:
1. **Strict-Transport-Security (HSTS):** Forces the browser to use HTTPS.
2. **X-Frame-Options:** Prevents the site from being loaded inside an `iframe`, protecting against Clickjacking.
3. **X-Content-Type-Options:** Prevents MIME-sniffing vulnerabilities.
4. **Content-Security-Policy (CSP):** Restricts where scripts and images can be loaded from, preventing Cross-Site Scripting (XSS).

---

## 2. Validating Headers using Python Requests

The easiest and fastest way to validate security headers is by completely bypassing the UI and using the `requests` library. This allows you to write lightning-fast infrastructure tests!

**tests/test_security_headers_api.py**

```python
import requests
def test_api_security_headers():
    # 1. Make an HTTP GET request to the environment
    response = requests.get("https://github.com")
    headers = response.headers
    print("\n[Security] Analyzing HTTP Response Headers...")
    # 2. Assert Strict-Transport-Security (HSTS)
    assert "Strict-Transport-Security" in headers, "HSTS Header is MISSING!"
    print(f"✅ HSTS: {headers['Strict-Transport-Security']}")
    # 3. Assert X-Frame-Options (Clickjacking Protection)
    assert "X-Frame-Options" in headers, "X-Frame-Options is MISSING!"
    print(f"✅ X-Frame-Options: {headers['X-Frame-Options']}")
    # 4. Assert X-Content-Type-Options
    assert "X-Content-Type-Options" in headers, "X-Content-Type-Options is MISSING!"
    print(f"✅ X-Content-Type-Options: {headers['X-Content-Type-Options']}")
    # 5. Assert Content-Security-Policy
    assert "Content-Security-Policy" in headers, "CSP Header is MISSING!"
    print(f"✅ CSP: {headers['Content-Security-Policy'][:50]}...") # Truncating for readability
```

---

## 3. Validating Headers using Selenium BiDi (CDP)

If you *must* validate the headers during an actual browser session (perhaps the headers change dynamically based on user interaction), you can use Selenium 4's Chrome DevTools Protocol (CDP) integration!

**tests/test_security_headers_ui.py**

```python
import time
from selenium import webdriver
def test_ui_security_headers():
    driver = webdriver.Chrome()
    # 1. Enable CDP Network Tracking
    driver.execute_cdp_cmd('Network.enable', {})
    security_headers = {}
    # 2. Define a listener to capture response headers
    def capture_headers(event):
        if "response" in event:
            url = event["response"]["url"]
            if url == "https://httpbin.org/get":
                print("\n[CDP] Captured network response for target URL!")
                headers = event["response"]["headers"]
                # Convert all keys to lower case for easy checking
                security_headers.update({k.lower(): v for k, v in headers.items()})
    driver.bidi_connection().session.execute(
        driver.bidi_connection().cdp.get_session_id(),
        "Network.responseReceived",
        capture_headers
    )
    # 3. Navigate to the page
    driver.get("https://httpbin.org/get")
    time.sleep(2) # Give CDP a moment to process the async event
    # Note: httpbin does NOT implement strong security headers, 
    # so we will check for standard headers instead to prove the concept.
    print("[Validation] Checking captured headers...")
    assert "content-type" in security_headers
    assert "access-control-allow-origin" in security_headers
    print("✅ Successfully verified headers directly from the browser's Network tab!")
    driver.quit()
```
*Note: Full network interception is much more reliable in Playwright or Cypress, but Selenium 4's CDP integration makes it possible!*

---

## 4. Execution Output

Let's execute the API-based security test against GitHub to see what Enterprise-grade security headers look like in the terminal!

```bash
pytest test_security_headers_api.py -s
```

```text
============================= test session starts ==============================
platform win32 -- Python 3.12.0, pytest-8.0.0
rootdir: C:\Automation\mycodeyatra-security
collected 1 item
test_security_headers_api.py 
[Security] Analyzing HTTP Response Headers...
✅ HSTS: max-age=31536000; includeSubdomains; preload
✅ X-Frame-Options: deny
✅ X-Content-Type-Options: nosniff
✅ CSP: default-src 'none'; base-uri 'self'; child-src o...
.
============================== 1 passed in 0.85s ===============================
```

## Conclusion

A single missing header can result in a devastating security breach. 
- Do not rely entirely on UI tests. 
- Use Python's `requests` library to build a fast "Infrastructure Sanity Suite" that explicitly checks for `Strict-Transport-Security`, `X-Frame-Options`, and `Content-Security-Policy`.
- Run these tests as the very first step in your CI/CD pipeline. If the headers are missing, fail the build immediately!

In our next article, we will dive deep into the most complex header of all: **The Content Security Policy (CSP)**. We will learn how to parse it, analyze it, and write automated tests to ensure no malicious domains are whitelisted!
