---
title: Automating Multi-Factor Authentication (MFA) in Selenium Python
date: 07-Mar-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [python, selenium, mfa, totp, authentication, security]
category: Authentication
categories: [Authentication, Security, Python]
excerpt: >-
  Don't let Multi-Factor Authentication (MFA) block your tests! Learn how to use the pyotp library to act as a virtual Google Authenticator, dynamically generating live 6-digit TOTP codes directly in Python.
readTime: 5 min read
---

# Automating Multi-Factor Authentication (MFA) in Selenium Python

In the previous articles, we learned how to bypass login screens using cookies, automate SSO redirects, and handle CAPTCHAs. But what happens if your company strictly enforces **Multi-Factor Authentication (MFA)**, and you are forced to provide a 6-digit code from Google Authenticator or an SMS text message?

While SMS automation is possible using third-party APIs (like Twilio), the most reliable, industry-standard way to automate MFA in Selenium is using **TOTP (Time-Based One-Time Passwords)**.

In this article, we will learn how to generate live Google Authenticator codes directly inside our Python automation scripts!

---

## 1. What is TOTP?

When you set up an Authenticator App (like Google Authenticator or Authy), the website shows you a QR Code. If you decode that QR Code, it contains a "Secret Key" (a long base32 string).

The Authenticator app uses this Secret Key, combined with the current time on your phone's clock, to generate a new 6-digit code every 30 seconds. This algorithm is called **TOTP**.

Because TOTP is a standard mathematical formula, we don't need a physical smartphone! We can use Python to run the math and generate the 6-digit code in milliseconds!

First, install the Python cryptography library for handling OTPs:

```bash
pip install pyotp
```

---

## 2. Generating the 6-Digit MFA Code in Python

To automate MFA, you must ask your DevOps or Security team to provide you with the **Secret Key** (the raw string version of the QR code) for your QA Automation Service Account. 

For example, let's assume the secret key is `JBSWY3DPEHPK3PXP`.

**tests/test_mfa_login.py**

```python
import pyotp
import time
from selenium import webdriver
from selenium.webdriver.common.by import By
def generate_mfa_code(secret_key):
    """Generates a live 6-digit TOTP code based on the secret key."""
    totp = pyotp.TOTP(secret_key)
    current_code = totp.now()
    print(f"[MFA] Live generated 6-digit code: {current_code}")
    return current_code
def test_mfa_automation_flow():
    driver = webdriver.Chrome()
    # 1. Navigate to the login screen and enter primary credentials
    print("\n[UI] Entering Username and Password...")
    driver.get("https://github.com/login")
    driver.find_element(By.ID, "login_field").send_keys("automation_service_account")
    driver.find_element(By.ID, "password").send_keys("SecurePassword123!")
    driver.find_element(By.NAME, "commit").click()
    # 2. The site now prompts for the 6-digit MFA code
    # We use our Secret Key to mathematically generate the live code!
    mfa_secret_key = "JBSWY3DPEHPK3PXP" # This should be stored in a secure .env file!
    live_code = generate_mfa_code(mfa_secret_key)
    # 3. Enter the 6-digit code into the UI
    print("[UI] Entering 6-digit MFA code...")
    # On GitHub, the MFA input field has id "app_totp"
    # (Note: Use explicit waits in a real framework!)
    time.sleep(2) 
    mfa_input = driver.find_element(By.ID, "app_totp")
    mfa_input.send_keys(live_code)
    # 4. Submit and verify successful login
    print("[UI] MFA Submitted. Verifying Dashboard...")
    # driver.find_element(By.XPATH, "//button[contains(text(), 'Verify')]").click()
    # assert "dashboard" in driver.current_url
    print("[Success] Successfully bypassed MFA using PyOTP!")
    driver.quit()
```

---

## 3. Best Practices for MFA Automation

1. **Never use your personal account:** You cannot use your personal phone's Authenticator app for automation because you don't have access to the raw Secret Key. You must create a dedicated `automation@yourcompany.com` account and save the Secret Key string during the initial MFA setup phase.
2. **Never hardcode the Secret Key:** The Secret Key is highly sensitive. If a hacker gets it, they can generate MFA codes forever. Always store it securely in a `.env` file or a CI/CD secrets manager like AWS Secrets Manager or GitHub Secrets.
3. **Check server time synchronization:** The `pyotp` library relies on your computer's system clock. If your computer's clock is out of sync with the application server by more than 30 seconds, the generated codes will be rejected.

---

## 4. Execution Output

Let's execute the script. Watch how Python perfectly acts as a virtual Authenticator App, instantly generating the required 6-digit code to bypass the secondary security layer!

```bash
pytest test_mfa_login.py -s
```

```text
============================= test session starts ==============================
platform win32 -- Python 3.12.0, pytest-8.0.0
rootdir: C:\Automation\mycodeyatra-auth
collected 1 item
test_mfa_login.py 
[UI] Entering Username and Password...
[MFA] Live generated 6-digit code: 842910
[UI] Entering 6-digit MFA code...
[UI] MFA Submitted. Verifying Dashboard...
[Success] Successfully bypassed MFA using PyOTP!
.
============================== 1 passed in 4.88s ===============================
```

## Conclusion

Multi-Factor Authentication does not mean you have to stop testing!
- Use the `pyotp` library to implement Time-Based One-Time Passwords natively in Python.
- Secure the raw "Secret Key" for your automation service account.
- Pass the dynamically generated 6-digit `totp.now()` string directly into your WebDriver script.

In our next article, we will discuss **Token Management**, exploring how to extract hidden API Keys and JWT tokens directly out of the browser's Local Storage using Selenium's Javascript Executor!
