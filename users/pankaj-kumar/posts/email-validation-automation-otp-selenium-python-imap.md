---
title: Email Validation: Automating OTPs and Verification Links
date: 07-Jun-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [python, selenium, email-validation, otp, imap, automation]
category: Enterprise Validation
categories: [Enterprise Validation, Python, Automation]
excerpt: >-
  Stop getting blocked by Registration flows! Learn how to use Python's built-in imaplib to programmatically access inboxes, extract OTPs via RegEx, and inject them back into your Selenium tests.
readTime: 6 min read
---

# Email Validation: Automating OTPs and Verification Links

One of the most common roadblocks for junior automation engineers is the dreaded "Email Verification" step. 

You write a perfect Selenium script to fill out a registration form. You click "Submit". And then the UI says: *"Please check your email for a 6-digit OTP."*

At this point, many engineers simply give up and ask developers to disable OTPs in the staging environment. But in a true Enterprise DevSecOps environment, you must test the application exactly as it behaves in production. 

In this article, we will teach you how to use Python's built-in `imaplib` to programmatically log into an email inbox, extract an OTP (or verification link), and seamlessly continue your Selenium test!

---

## 1. Setting Up Your Testing Email

To automate email reading, you cannot use your personal Gmail account with Two-Factor Authentication enabled. You need a dedicated testing inbox that supports basic IMAP access or App Passwords.

**Best Practices for Test Emails:**
1. Use an **App Password** for Gmail (if Google Workspace allows it).
2. Use a dedicated email testing service like **Mailtrap**, **Mailosaur**, or **Guerrilla Mail**.
3. In this example, we will assume you have a Gmail account with an App Password.

---

## 2. Writing the Email Extraction Utility

Python comes with `imaplib` and `email` modules built directly into the standard library. We do not need to `pip install` anything!

Let's write a utility function that logs into the inbox, finds the most recent email with a specific subject line, and uses a Regular Expression (RegEx) to extract a 6-digit OTP from the email body.

**utils/email_helper.py**

```python
import imaplib
import email
import re
import time
def extract_otp_from_email(username, app_password, subject_filter="Your Verification Code"):
    # Connect to the Gmail IMAP server
    mail = imaplib.IMAP4_SSL('imap.gmail.com')
    # Login
    mail.login(username, app_password)
    # Select the Inbox
    mail.select('inbox')
    print(f"\n[Email] Waiting for OTP email with subject: '{subject_filter}'...")
    # Wait up to 30 seconds for the email to arrive
    for i in range(15):
        # Search for all unread emails with the specific subject
        status, messages = mail.search(None, f'(UNSEEN SUBJECT "{subject_filter}")')
        email_ids = messages[0].split()
        if email_ids:
            # Get the most recent email
            latest_email_id = email_ids[-1]
            # Fetch the email contents
            status, msg_data = mail.fetch(latest_email_id, '(RFC822)')
            raw_email = msg_data[0][1]
            msg = email.message_from_bytes(raw_email)
            # Extract the body text
            body = ""
            if msg.is_multipart():
                for part in msg.walk():
                    if part.get_content_type() == "text/plain":
                        body = part.get_payload(decode=True).decode()
            else:
                body = msg.get_payload(decode=True).decode()
            # Use RegEx to find exactly 6 digits!
            match = re.search(r'\b\d{6}\b', body)
            if match:
                otp = match.group(0)
                print(f"[Email] Successfully extracted OTP: {otp}")
                # Close connection and return
                mail.close()
                mail.logout()
                return otp
        # If no email found, wait 2 seconds and try again
        time.sleep(2)
    mail.close()
    mail.logout()
    raise Exception("OTP Email never arrived!")
```

---

## 3. The Hybrid Test: Selenium + IMAP Automation

Now, let's tie it all together in our Pytest framework. We will use Selenium to trigger the OTP email, pause the UI to run our `imaplib` script, and then inject the extracted OTP back into the browser!

**tests/test_registration.py**

```python
from selenium.webdriver.common.by import By
from utils.email_helper import extract_otp_from_email
import os
def test_registration_with_otp_verification(driver):
    # 1. UI ACTION: Trigger the OTP
    driver.get("https://mycodeyatra.com/register")
    test_email = os.getenv("TEST_EMAIL_ADDRESS")
    test_password = os.getenv("TEST_EMAIL_APP_PASS")
    driver.find_element(By.ID, "username").send_keys("automation_expert")
    driver.find_element(By.ID, "email").send_keys(test_email)
    driver.find_element(By.ID, "register-btn").click()
    # The UI now shows the OTP input field
    assert driver.find_element(By.ID, "otp-input").is_displayed()
    # 2. EMAIL AUTOMATION: Go fetch the OTP from the backend inbox!
    extracted_otp = extract_otp_from_email(
        username=test_email, 
        app_password=test_password, 
        subject_filter="Verify Your MyCodeYatra Account"
    )
    # 3. UI ACTION: Enter the extracted OTP and submit
    driver.find_element(By.ID, "otp-input").send_keys(extracted_otp)
    driver.find_element(By.ID, "verify-btn").click()
    # 4. FINAL ASSERTION
    success_text = driver.find_element(By.ID, "dashboard-header").text
    assert "Welcome to your Dashboard" in success_text
```

### Extracting Links Instead of OTPs
If your application sends a verification *link* instead of an OTP, simply update the Regular Expression in your utility function to search for URLs (`r'https?://[^\s]+'`), return the URL, and use `driver.get(extracted_url)` to navigate to it!

## Conclusion

Never let an Email Verification step stop your test automation suite again!
- Avoid personal accounts; use dedicated Testing Inboxes with App Passwords.
- Use Python's native `imaplib` to connect to the email server.
- Use `mail.search()` to filter for unread messages with specific subjects.
- Use the `re` module (Regular Expressions) to parse the email body and extract the 6-digit OTP or URL.
- Pass the extracted data back to your Selenium `driver` to complete the UI workflow!

In our next article, we will look at how to automate file downloads and validate the contents of **PDFs and Excel Files** natively in Python!
