---
title: Handling Frames and iFrames in Selenium Python
date: 15-Dec-2024
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [selenium, python, iframes, frames, switch-to-frame, testing]
category: UI Automation
categories: [UI Automation, Selenium, Python]
excerpt: >-
  If Selenium can't find an element, it might be hiding in an iframe! Learn how to switch contexts into iframes, handle nested iframes, and return to the main document.
readTime: 5 min read
---

# Handling Frames and iFrames in Selenium Python

In modern web development, embedding external content is extremely common. If a website displays a YouTube video, a Stripe payment gateway, or a Google Map, it is almost certainly doing so using an **`<iframe>`** (Inline Frame).

An `<iframe>` is essentially a complete HTML document embedded *inside* another HTML document. 

Just like with multiple tabs, Selenium can only see the DOM of the document it is currently focused on. If an element exists inside an iframe, Selenium will completely ignore it and throw a `NoSuchElementException`!

In this article, we will learn how to detect iframes, switch our driver context *into* them, and switch back out.

---

## 1. Switching into an iFrame

To interact with an element inside an iframe, you must explicitly tell Selenium to step inside it. There are three ways to switch into an iframe using `driver.switch_to.frame()`:

1. **By Index** (Zero-based integer)
2. **By Name or ID** (String)
3. **By WebElement** (Locating it like any other element)

```python
from selenium import webdriver
from selenium.webdriver.common.by import By
def test_switch_to_iframe():
    driver = webdriver.Chrome()
    driver.get("https://mycodeyatra.com/payments")
    # Let's say there is an iframe rendering a Stripe credit card input
    # Strategy 1: Switch by Index (Not recommended if the page has dynamic iframes)
    # driver.switch_to.frame(0)
    # Strategy 2: Switch by Name or ID attribute of the <iframe> tag
    # <iframe id="stripe-checkout" ...>
    # driver.switch_to.frame("stripe-checkout")
    # Strategy 3: Switch by WebElement (The most robust method!)
    iframe_element = driver.find_element(By.CSS_SELECTOR, "iframe.payment-gateway")
    driver.switch_to.frame(iframe_element)
    # Now that we are INSIDE the iframe, we can find elements inside it!
    cc_input = driver.find_element(By.ID, "card-number")
    cc_input.send_keys("4242 4242 4242 4242")
    assert cc_input.get_attribute("value") == "4242 4242 4242 4242"
    driver.quit()
```

---

## 2. Returning to the Parent Document

Once you enter an iframe, you are trapped there. If you try to interact with the main website's header or footer, Selenium will fail because those elements exist in the Parent DOM.

To get back to the main website, you must use `default_content()`.

```python
def test_iframe_return_to_parent():
    driver = webdriver.Chrome()
    driver.get("https://mycodeyatra.com/payments")
    # 1. Switch INTO the iframe
    driver.switch_to.frame("stripe-checkout")
    # 2. Interact with the iframe
    driver.find_element(By.ID, "submit-payment").click()
    # 3. Step OUT of the iframe back to the main document
    driver.switch_to.default_content()
    # 4. Now we can interact with the main page again!
    success_toast = driver.find_element(By.ID, "main-page-toast")
    assert success_toast.text == "Payment processing..."
    driver.quit()
```

---

## 3. Handling Nested iFrames

Sometimes, the internet gets weird. You might encounter an iframe *inside* an iframe (e.g., a widget inside an embedded dashboard).

To handle nested iframes, you must step into them one by one like a ladder. You cannot jump straight to the inner iframe!

```python
def test_nested_iframes():
    driver = webdriver.Chrome()
    driver.get("https://mycodeyatra.com/nested")
    # Step 1: Switch to the Outer iFrame
    driver.switch_to.frame("outer-dashboard-frame")
    # Step 2: Switch to the Inner iFrame (which lives inside the outer frame)
    driver.switch_to.frame("inner-widget-frame")
    # Step 3: Interact with the innermost element
    inner_btn = driver.find_element(By.ID, "widget-btn")
    inner_btn.click()
    # How do we get out?
    # Option A: Go all the way back to the very top (Main Document)
    # driver.switch_to.default_content()
    # Option B: Step exactly one level up to the Outer iFrame
    driver.switch_to.parent_frame()
    # Now we are in the Outer iFrame!
    outer_header = driver.find_element(By.TAG_NAME, "h2").text
    assert outer_header == "Dashboard Overview"
    driver.quit()
```

### `default_content()` vs `parent_frame()`
- `default_content()`: An elevator that takes you straight to the ground floor (Main HTML document).
- `parent_frame()`: Stairs that take you exactly one level up.

---

## 4. Execution Output

When we run these scenarios via PyTest, the context-switching happens seamlessly behind the scenes!

```text
============================= test session starts ==============================
platform win32 -- Python 3.12.0, pytest-8.0.0
rootdir: C:\Automation\mycodeyatra-tests
collected 3 items
test_iframes.py::test_switch_to_iframe PASSED                            [ 33%]
test_iframes.py::test_iframe_return_to_parent PASSED                     [ 66%]
test_iframes.py::test_nested_iframes PASSED                              [100%]
============================== 3 passed in 12.04s ==============================
```

## Conclusion

Whenever your Selenium script fails to find an element that you can clearly see on the screen, your first instinct should be: **"Is this inside an iframe?"**

- Right-click the element and check if "View Frame Source" is an option. If it is, you need to use `switch_to.frame()`.
- Always remember to use `switch_to.default_content()` when you are done!

In our next article, we will look at an even more complex isolation boundary: The **Shadow DOM**. See you there!
