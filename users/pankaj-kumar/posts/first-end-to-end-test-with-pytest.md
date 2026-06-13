---
title: First E2E Test with PyTest: Putting It All Together
date: 21-Nov-2024
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [selenium, python, pytest, e2e, reporting, pytest-html]
category: UI Automation
categories: [UI Automation, Selenium, Python]
excerpt: >-
  Write your first complete End-to-End UI test in Python. Combine WebDriver initialization, element locators, and PyTest assertions, and generate a beautiful HTML report.
readTime: 5 min read
---

# First E2E Test with PyTest: Putting It All Together

Over the last few articles, we have learned how to isolate dependencies, initialize the browser via the Native Selenium Manager, locate elements on the DOM, and validate expected outcomes using PyTest assertions.

It is time to put all of these pieces together.

In this article, we will write our very first complete **End-to-End (E2E) Test** against our live practice site, `https://mycodeyatra.com`. We will test the login functionality from start to finish, and we will generate a beautiful HTML report.

---

## 1. The Scenario

Our test case is simple but covers all the critical aspects of UI automation:
1. **Initialize** the Chrome Browser.
2. **Navigate** to `https://mycodeyatra.com/login`.
3. **Locate** the Email and Password fields.
4. **Interact** by typing in dummy credentials and clicking Submit.
5. **Assert** that the URL changed to `/dashboard` and the Welcome banner is displayed.
6. **Teardown** the browser gracefully to avoid memory leaks.

---

## 2. Writing the PyTest Script

Create a new file named `test_login.py`. Remember, `pytest` will automatically discover any file that starts with `test_`!

```python
import time
from selenium import webdriver
from selenium.webdriver.common.by import By
def test_successful_login():
    # 1. INITIALIZATION
    print("\nStarting Login Test...")
    driver = webdriver.Chrome()
    driver.maximize_window()
    # Implicit Wait (We will discuss waits in-depth in Phase 2!)
    driver.implicitly_wait(10)
    try:
        # 2. NAVIGATION
        driver.get("https://mycodeyatra.com/login")
        # 3 & 4. LOCATE AND INTERACT
        email_field = driver.find_element(By.ID, "email")
        email_field.send_keys("admin@mycodeyatra.com")
        password_field = driver.find_element(By.ID, "password")
        password_field.send_keys("SuperSecret123!")
        login_btn = driver.find_element(By.CSS_SELECTOR, "button[type='submit']")
        login_btn.click()
        # Wait a moment for navigation
        time.sleep(2)
        # 5. ASSERTIONS
        # Hard assertion: Did the URL change?
        assert "dashboard" in driver.current_url, "Login failed: URL did not redirect to dashboard!"
        # Hard assertion: Is the welcome text correct?
        welcome_text = driver.find_element(By.TAG_NAME, "h1").text
        assert "Welcome back" in welcome_text, f"Expected 'Welcome back', but got: {welcome_text}"
        print("Login test passed successfully!")
    finally:
        # 6. TEARDOWN (Guarantees execution even if assertions fail)
        driver.quit()

```

---

## 3. Configuring PyTest

Before we run this, let's configure `pytest` so the console output looks professional.

Create a file named `pytest.ini` in the root of your project:

```ini
[pytest]
addopts = -v -s
testpaths = tests

```
- `-v` stands for "verbose", giving us a detailed breakdown of which tests passed.
- `-s` tells pytest to print our console logs (like `print("\nStarting Login Test...")`) instead of hiding them.

---

## 4. Running the Test and Generating a Report

In a professional environment, running tests in the console isn't enough. Your managers will want a report. Earlier, we installed `pytest-html` via our `requirements.txt`.

Let's use it! Open your terminal and run:

```bash
pytest test_login.py --html=report.html

```

### What happens?
1. PyTest detects `test_login.py` and executes `test_successful_login`.
2. Chrome launches automatically (thanks to Selenium Manager).
3. The script drives the browser, types the text, and clicks Login.
4. The assertions pass, and the `finally` block kills the Chrome process.
5. PyTest generates a beautiful `report.html` file in your directory!

Open `report.html` in your browser. You will see a detailed, color-coded summary of your test execution!

## Conclusion

Congratulations! You have officially written a complete, robust Selenium Python test. 

However, looking at our code, you might notice something: our test method contains initialization code, teardown code, and test logic all mashed together. If we write 50 tests, we will duplicate that `webdriver.Chrome()` and `driver.quit()` logic 50 times.

In Phase 2 of our curriculum, we will solve this architectural problem by diving into **PyTest Fixtures** and the **Page Object Model**!
