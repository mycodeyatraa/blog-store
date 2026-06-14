---
title: Multi-User Testing in Selenium Python: Admin vs Customer Roles
date: 28-Feb-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [python, selenium, multi-user, roles, webdriver, architecture]
category: Authentication
categories: [Authentication, Security, Python]
excerpt: >-
  Test complex B2B workflows! Learn how to launch multiple isolated WebDriver sessions simultaneously in Python to test real-time interactions between Admin and Customer roles.
readTime: 6 min read
---

# Multi-User Testing in Selenium Python: Admin vs Customer Roles

In standard UI test automation, a test script usually mimics a single user. You launch the browser, log in as a Customer, place an order, and close the browser.

But what if you are testing a complex B2B workflow? 
1. The **Customer** submits a ticket.
2. The **Admin** must log in to approve the ticket.
3. The **Customer** receives an approval notification.

If you test this sequentially (login as customer -> logout -> login as admin -> logout -> login as customer), your test will be painfully slow and flaky. Instead, we can implement **Multi-User Testing** by launching two entirely separate, isolated WebDriver instances in the exact same Python script!

---

## 1. Launching Multiple WebDriver Instances

Selenium allows you to instantiate as many `webdriver` objects as your machine's RAM can handle. Because each instance gets its own temporary profile, they do not share cookies. This means you can log into the exact same application with two different accounts simultaneously.

Let's write a script that opens two browsers side-by-side: one for the Customer, and one for the Admin.

**tests/test_multi_user.py**

```python
import time
from selenium import webdriver
def test_admin_and_customer_simultaneously():
    # 1. Launch the Customer Browser
    customer_driver = webdriver.Chrome()
    # 2. Launch the Admin Browser
    admin_driver = webdriver.Chrome()
    # 3. Position the windows side-by-side so we can watch them!
    customer_driver.set_window_rect(x=0, y=0, width=960, height=1080)
    admin_driver.set_window_rect(x=960, y=0, width=960, height=1080)
    print("\n[Multi-User] Both browsers launched successfully!")
    # 4. Navigate both browsers independently
    customer_driver.get("https://httpbin.org/basic-auth/customer/pass")
    admin_driver.get("https://httpbin.org/basic-auth/admin/pass")
    # Because httpbin uses basic auth, the URL will trigger a 401 prompt.
    # In a real app, you would execute the login flow for both drivers here.
    print(f"[Customer Browser] URL: {customer_driver.current_url}")
    print(f"[Admin Browser] URL: {admin_driver.current_url}")
    # 5. Clean up both drivers!
    time.sleep(2)
    customer_driver.quit()
    admin_driver.quit()
```

---

## 2. Using Python Dictionaries to Manage Drivers

If you have a scenario requiring 3 or 4 users (e.g., Buyer, Seller, Escrow Agent, Admin), managing `driver_1`, `driver_2`, `driver_3` becomes messy. 

Enterprise SDETs store their WebDrivers inside a Python Dictionary. This allows you to easily switch contexts by referencing the user's role!

```python
def test_driver_dictionary_management():
    # Store drivers in a dictionary mapped to their role
    sessions = {
        "Buyer": webdriver.Chrome(),
        "Seller": webdriver.Chrome()
    }
    print(f"\n[Manager] Active Sessions: {list(sessions.keys())}")
    # Buyer Workflow
    sessions["Buyer"].get("https://mycodeyatra.com")
    print(f"[Buyer] Currently looking at: {sessions['Buyer'].title}")
    # Seller Workflow
    sessions["Seller"].get("https://github.com")
    print(f"[Seller] Currently looking at: {sessions['Seller'].title}")
    # Clean up all sessions dynamically
    for role, driver in sessions.items():
        print(f"Closing {role} session...")
        driver.quit()
```

---

## 3. Execution Output

Let's execute both tests. Notice how Python effortlessly handles two totally independent Chromium processes, passing commands to each specific role without any data leakage!

```bash
pytest test_multi_user.py -s
```

```text
============================= test session starts ==============================
platform win32 -- Python 3.12.0, pytest-8.0.0
rootdir: C:\Automation\mycodeyatra-auth
collected 2 items
test_multi_user.py 
[Multi-User] Both browsers launched successfully!
[Customer Browser] URL: https://httpbin.org/basic-auth/customer/pass
[Admin Browser] URL: https://httpbin.org/basic-auth/admin/pass
.
[Manager] Active Sessions: ['Buyer', 'Seller']
[Buyer] Currently looking at: MyCodeYatra - Learn Test Automation
[Seller] Currently looking at: GitHub: Let's build from here
Closing Buyer session...
Closing Seller session...
.
============================== 2 passed in 12.35s ==============================
```

## Conclusion

Testing multi-actor workflows doesn't require complex parallel execution frameworks.
- You can instantiate multiple `webdriver.Chrome()` objects in the same test script.
- Since each driver uses a fresh profile, they have completely isolated cookies and local storage.
- Always store multiple drivers in a Python Dictionary (e.g., `drivers["Admin"]`) to keep your code readable and scalable.
- Remember to call `.quit()` on *all* driver instances at the end of the test!

In our next article, we will tackle the holy grail of Enterprise security: **Single Sign-On (SSO) Authentication!** We will learn how to bypass Microsoft Azure AD and Okta login gateways using Selenium.
