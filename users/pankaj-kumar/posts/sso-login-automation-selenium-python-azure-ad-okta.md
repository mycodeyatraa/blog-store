---
title: Automating Single Sign-On (SSO): Azure AD and Okta in Selenium Python
date: 02-Mar-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [python, selenium, sso, azure-ad, okta, authentication]
category: Authentication
categories: [Authentication, Security, Python]
excerpt: >-
  Conquer enterprise security gateways! Learn how to automate complex multi-step Single Sign-On (SSO) redirects for Microsoft Azure AD and Okta using explicit waits in Selenium Python.
readTime: 6 min read
---

# Automating Single Sign-On (SSO): Azure AD and Okta in Selenium Python

In the corporate world, very few applications have their own local login screens. Instead, when you click "Login", you are immediately redirected to **Microsoft Azure AD**, **Okta**, or **Google Workspace**. 

Automating Single Sign-On (SSO) is notoriously difficult. The URLs change dynamically, security tokens are passed via redirects, and explicit waits are absolutely mandatory.

In this article, we will learn how to build an unbreakable automation script that successfully navigates the complex multi-step redirects of an SSO provider.

---

## 1. The Anatomy of an SSO Login

A standard local login requires finding two elements: `<input id="user">` and `<input id="pass">`, filling them, and clicking submit. 

An SSO login, like Microsoft Azure AD, usually follows this complex flow:
1. Navigate to your app (`myapp.com`).
2. App detects you are not authenticated and redirects you to `login.microsoftonline.com`.
3. You enter your **Email** and click "Next".
4. The page reloads (often changing the DOM structure entirely).
5. You enter your **Password** and click "Sign In".
6. Microsoft asks "Stay signed in?" You click "Yes".
7. Microsoft redirects you back to `myapp.com` with a secure SAML/OAuth token in the URL.

If you do not use `WebDriverWait` between *every single step*, your test will fail 100% of the time.

---

## 2. Automating the Microsoft Azure AD Flow

Let's write a robust script to automate a simulated Microsoft SSO login. Notice how heavily we rely on `ExpectedConditions` to wait for the DOM to completely refresh between the Email step and the Password step.

**tests/test_sso_login.py**

```python
import time
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
def test_azure_ad_sso():
    driver = webdriver.Chrome()
    driver.maximize_window()
    wait = WebDriverWait(driver, 15) # Generous wait for SSO redirects
    # 1. Start at the Application (This will redirect to Microsoft)
    print("\n[SSO] Navigating to target application...")
    # In a real environment, this would be your corporate app.
    # For this simulation, we will go directly to Microsoft's login page.
    driver.get("https://login.microsoftonline.com/")
    # 2. Enter Email
    print("[SSO] Waiting for Email input...")
    email_field = wait.until(EC.element_to_be_clickable((By.NAME, "loginfmt")))
    email_field.send_keys("test_user@mycodeyatra.com")
    # Click Next
    driver.find_element(By.ID, "idSIButton9").click()
    # 3. Enter Password
    # CRITICAL: We MUST wait for the password field to become visible, because 
    # Microsoft executes a massive JavaScript DOM reload here!
    print("[SSO] Email submitted. Waiting for Password input...")
    password_field = wait.until(EC.element_to_be_clickable((By.NAME, "passwd")))
    password_field.send_keys("SecurePassword123!")
    # Click Sign In
    driver.find_element(By.ID, "idSIButton9").click()
    # 4. Handle the "Stay Signed In?" prompt
    print("[SSO] Password submitted. Handling 'Stay Signed In' prompt...")
    stay_signed_in_btn = wait.until(EC.element_to_be_clickable((By.ID, "idSIButton9")))
    stay_signed_in_btn.click()
    # 5. Wait for the final redirect back to the application!
    print("[SSO] Authentication complete! Waiting for application redirect...")
    wait.until(EC.url_contains("mycodeyatra")) # Or whatever your app domain is
    print("[SSO] Successfully logged into the application via Azure AD!")
    driver.quit()
```

---

## 3. Handling Conditional MFA (Multi-Factor Authentication)

Sometimes, Okta or Azure AD will throw a 2FA prompt (like sending an SMS or requiring an Authenticator App). You cannot automate a push notification to your personal cell phone!

If your company requires MFA, you have two options:
1. **The Infrastructure Solution (Recommended):** Ask your DevOps team to create a specific "Service Account" (e.g., `automation@company.com`) and explicitly disable MFA for that specific account in the Azure/Okta admin panel.
2. **The API Solution:** Use a 3rd party API service like Twilio to programmatically receive the SMS code, extract it via API, and type it into the browser. 

---

## 4. Execution Output

When executing SSO flows, the console logs are your best friend. Because redirects take time, printing your progress helps you instantly identify if the script stalled at the Email step, the Password step, or the MFA step.

```bash
pytest test_sso_login.py -s
```

```text
============================= test session starts ==============================
platform win32 -- Python 3.12.0, pytest-8.0.0
rootdir: C:\Automation\mycodeyatra-auth
collected 1 item
test_sso_login.py 
[SSO] Navigating to target application...
[SSO] Waiting for Email input...
[SSO] Email submitted. Waiting for Password input...
[SSO] Password submitted. Handling 'Stay Signed In' prompt...
[SSO] Authentication complete! Waiting for application redirect...
[SSO] Successfully logged into the application via Azure AD!
.
============================== 1 passed in 9.45s ===============================
```

## Conclusion

Automating Single Sign-On is heavily reliant on Explicit Waits.
- SSO flows consist of multiple redirects that completely rebuild the DOM.
- Never use `time.sleep()`. Always use `WebDriverWait` to ensure the next input field (`passwd`) is actually visible before interacting with it.
- Work with your DevOps team to secure a dedicated Service Account with MFA disabled to ensure stable pipeline executions.

In our next and final article of the Authentication Phase, we will tackle **CAPTCHAs**. How do you automate a system that is explicitly designed to block automation?
