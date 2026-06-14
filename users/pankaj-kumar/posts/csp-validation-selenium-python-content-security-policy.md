---
title: Validating Content Security Policy (CSP) with Python Automation
date: 20-Mar-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [python, selenium, security, csp, headers, xss]
category: Security
categories: [Security, Python, DevSecOps]
excerpt: >-
  Stop Cross-Site Scripting (XSS) at the pipeline! Learn how to parse Content Security Policy (CSP) headers in Python and write automated tests to mathematically prove your application blocks malicious domains.
readTime: 6 min read
---

# Validating Content Security Policy (CSP) with Python Automation

In our previous article, we learned how to check if the `Content-Security-Policy` header simply exists. But checking for its *existence* is not enough. 

The Content Security Policy (CSP) is a massive string of rules that dictates exactly which domains are permitted to run JavaScript, load images, or embed frames on your website. If a developer misconfigures the CSP and accidentally includes `unsafe-inline` or a wildcard `*`, your application instantly becomes vulnerable to **Cross-Site Scripting (XSS)**.

In this article, we will write a Python script that parses the raw CSP header, converts it into a searchable dictionary, and rigorously validates the security rules!

---

## 1. What Does a CSP Look Like?

A standard CSP header looks like a long string separated by semicolons:
`default-src 'self'; script-src 'self' https://trusted-cdn.com; img-src *; frame-ancestors 'none'`

Each chunk is called a **directive**. 
- `default-src 'self'`: Only load assets from the same domain by default.
- `script-src`: Overrides `default-src` specifically for JavaScript. Here it allows `'self'` and a trusted CDN.
- `img-src *`: Overrides `default-src` for images. Here, the wildcard `*` allows images from anywhere.
- `frame-ancestors 'none'`: Prevents the site from being loaded inside an iframe (similar to X-Frame-Options).

---

## 2. Parsing the CSP in Python

To automate our validation, we first need to fetch the CSP header and parse it into a Python Dictionary.

**tests/test_csp_validation.py**

```python
import requests
import pytest
def get_parsed_csp(url: str) -> dict:
    """Fetches the CSP header and parses it into a dictionary."""
    response = requests.get(url)
    # 1. Ensure the header exists
    csp_raw = response.headers.get("Content-Security-Policy")
    assert csp_raw is not None, f"CSP header missing on {url}"
    # 2. Parse the string into a dictionary
    csp_dict = {}
    # Split by semicolon to get directives
    directives = [d.strip() for d in csp_raw.split(";") if d.strip()]
    for directive in directives:
        # The first word is the key, the rest are values
        parts = directive.split(" ")
        key = parts[0]
        values = parts[1:] if len(parts) > 1 else []
        csp_dict[key] = values
    return csp_dict
```

---

## 3. Writing CSP Security Tests

Now that we have the CSP formatted as a beautiful Python Dictionary (e.g., `{"script-src": ["'self'", "https://cdn.com"]}`), we can write targeted security assertions using Pytest!

We want to enforce the following enterprise rules:
1. `unsafe-inline` must NEVER be used in `script-src` (it allows raw `<script>` tags).
2. The wildcard `*` must NEVER be used in `script-src` (it allows JS from any malicious domain).
3. `frame-ancestors` must be `'none'` or `'self'` to prevent Clickjacking.

Let's append these tests to our file:

```python
# Assuming we are testing GitHub's highly secure CSP
TARGET_URL = "https://github.com"
def test_csp_no_unsafe_inline():
    csp = get_parsed_csp(TARGET_URL)
    # Check if script-src is explicitly defined, otherwise fallback to default-src
    scripts = csp.get("script-src", csp.get("default-src", []))
    assert "'unsafe-inline'" not in scripts, "CRITICAL VULNERABILITY: 'unsafe-inline' is enabled!"
    print(f"\n✅ Passed: 'unsafe-inline' is successfully blocked.")
def test_csp_no_wildcard_scripts():
    csp = get_parsed_csp(TARGET_URL)
    scripts = csp.get("script-src", csp.get("default-src", []))
    assert "*" not in scripts, "CRITICAL VULNERABILITY: Wildcard '*' script loading is enabled!"
    print(f"✅ Passed: Wildcard script execution is blocked.")
def test_csp_frame_ancestors():
    csp = get_parsed_csp(TARGET_URL)
    # Check if frame-ancestors is defined to protect against Clickjacking
    assert "frame-ancestors" in csp, "frame-ancestors directive is completely missing!"
    allowed = csp["frame-ancestors"]
    assert "'none'" in allowed or "'self'" in allowed, "frame-ancestors allows untrusted framing!"
    print(f"✅ Passed: Clickjacking explicitly prevented via frame-ancestors.")
```

---

## 4. Execution Output

Let's execute our new CSP test suite against GitHub. Watch as Python mathematically proves that the site's security architecture is robust!

```bash
pytest test_csp_validation.py -s
```

```text
============================= test session starts ==============================
platform win32 -- Python 3.12.0, pytest-8.0.0
rootdir: C:\Automation\mycodeyatra-security
collected 3 items
test_csp_validation.py 
✅ Passed: 'unsafe-inline' is successfully blocked.
.
✅ Passed: Wildcard script execution is blocked.
.
✅ Passed: Clickjacking explicitly prevented via frame-ancestors.
.
============================== 3 passed in 1.45s ===============================
```

## Conclusion

A Content Security Policy is your application's absolute last line of defense against Cross-Site Scripting (XSS). 
- Do not let developers sneak `'unsafe-inline'` into the CSP just to fix a broken library.
- By parsing the CSP header into a Python dictionary, you can write targeted, automated pipeline tests that fail the deployment if the security policy is weakened.

In our next article, we will take a step back from HTTP headers and look at JWTs. We will learn how to decode and validate **JSON Web Tokens (JWTs)** directly in Python to ensure no sensitive passwords are leaked inside the payload!
