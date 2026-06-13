---
title: Assertions with PyTest: Validating Test Outcomes
date: 18-Nov-2024
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [selenium, python, pytest, assertions, pytest-check]
category: UI Automation
categories: [UI Automation, Selenium, Python]
excerpt: >-
  Transform your scripts into actual tests! Discover how PyTest's assertion introspection makes validations incredibly clean, and learn how to implement Soft Assertions using pytest-check.
readTime: 5 min read
---

# Assertions with PyTest: Validating Test Outcomes

So far, we have written code that launches a browser, navigates to `https://mycodeyatra.com`, and locates a button. 

But navigating and clicking does not mean you have written an Automated Test. If a script clicks "Login" and the screen turns completely blank, Selenium won't care. It successfully executed the click, so the script will silently pass!

To convert a script into a **Test**, we must define expected outcomes and compare them against actual outcomes. We do this using **Assertions**. 

In Python, the `pytest` framework completely revolutionizes how we write assertions compared to legacy frameworks like Java's TestNG or JUnit.

---

## 1. The Magic of the PyTest `assert` Keyword

In Java or C#, if you want to assert that two values are equal, you have to import a specific assertion class and use a verbose method:
```java
// Java TestNG example
Assert.assertEquals(actualTitle, "MyCodeYatra", "Title did not match!");
Assert.assertTrue(button.isDisplayed(), "Button is hidden!");
```

PyTest completely eliminates this boilerplate. Instead of providing 50 different assertion methods (`assertEquals`, `assertTrue`, `assertNull`), PyTest hooks directly into Python's native `assert` keyword!

```python
# Python PyTest example
def test_dashboard_title():
    actual_title = driver.title
    # Simple, readable, native Python
    assert actual_title == "MyCodeYatra Dashboard", "Title did not match!"
```

### How does PyTest do this? (Assertion Introspection)
If a native Python `assert` fails, it normally just throws an ugly `AssertionError` with zero context. But PyTest intercepts the code before it runs and **rewrites the AST (Abstract Syntax Tree)**. 

If the assertion above fails, PyTest prints a beautiful diff in the console, showing exactly what `actual_title` was, what you expected it to be, and highlighting the exact character where the strings diverged!

---

## 2. Common Selenium Assertions in PyTest

Because you are just using native Python, asserting UI states becomes incredibly intuitive:

```python
from selenium.webdriver.common.by import By
def test_login_page_elements():
    # 1. Asserting Text Equality
    header = driver.find_element(By.TAG_NAME, "h1")
    assert header.text == "Secure Login"
    # 2. Asserting a Boolean State (Is the element displayed?)
    error_msg = driver.find_element(By.ID, "error-toast")
    assert error_msg.is_displayed() is False, "Error message should be hidden!"
    # 3. Asserting Substrings (Contains)
    current_url = driver.current_url
    assert "dashboard" in current_url, f"Expected dashboard in URL, but got {current_url}"
    # 4. Asserting List Sizes (Did our search return 5 items?)
    product_cards = driver.find_elements(By.CSS_SELECTOR, ".product-card")
    assert len(product_cards) == 5, "Expected exactly 5 products on the page"
```

---

## 3. Hard vs. Soft Assertions

The `assert` keyword in Python is a **Hard Assertion**. 
This means the absolute millisecond the assertion fails, an Exception is thrown, the test execution aborts immediately, and the remaining code in that function is skipped.

But what if you are validating a user profile page with 10 different fields (First Name, Last Name, Email, Phone)? If the First Name is wrong, a Hard Assertion will abort the test, meaning you have no idea if the other 9 fields were correct or not!

### Introducing Soft Assertions
A **Soft Assertion** logs the failure, but allows the test to continue executing. At the very end of the test, it tallies up all the failures and fails the test.

To do this in PyTest, we use an amazing plugin called `pytest-check`.

First, install it:
```bash
pip install pytest-check
```

Now, implement it in your test:
```python
import pytest_check as check
def test_user_profile():
    # ... navigation code ...
    # Even if 'first_name' fails, the test will NOT stop!
    check.equal(first_name.text, "John", "First name mismatch")
    # It will continue and check the last name
    check.equal(last_name.text, "Doe", "Last name mismatch")
    # It will continue and check the email
    check.is_in("@mycodeyatra.com", email.text, "Invalid email domain")
    # At the end of the function, pytest-check will fail the test and report ALL 3 errors at once!
```

## Conclusion

Assertions are the judge and jury of your automation framework. Thanks to PyTest's assertion introspection, writing validations in Python is significantly faster and cleaner than any other language.

By mastering native `assert` for critical validations (like verifying a login succeeded) and `pytest-check` for mass-data validations (like verifying a profile page), your test reports will become incredibly precise.

Now that we know how to navigate, locate elements, and assert outcomes, we finally have all the pieces of the puzzle. In our next article, we will combine everything and write our **First End-to-End Test with PyTest!**
