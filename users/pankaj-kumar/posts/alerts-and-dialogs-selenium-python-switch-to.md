---
title: Handling Alerts and Dialogs in Selenium Python
date: 05-Dec-2024
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [selenium, python, alerts, dialogs, popups, switch-to]
category: UI Automation
categories: [UI Automation, Selenium, Python]
excerpt: >-
  Conquer native browser popups. Learn how to switch context and interact with Alerts, Confirms, and Prompts in Selenium Python, and how to wait for them explicitly.
readTime: 5 min read
---

# Handling Alerts and Dialogs in Selenium Python

While navigating web applications, you have likely encountered native browser popups: simple dialog boxes that ask you to click "OK" or "Cancel", or even prompt you to enter some text.

These are native **JavaScript Alerts**. They are rendered directly by the browser (Chrome, Firefox, etc.) and are completely separate from the HTML DOM. 

Because they are not part of the DOM, you **cannot** right-click and inspect them. You cannot find them using `By.ID` or `By.XPATH`. If you try to interact with the webpage while an Alert is open, Selenium will crash with an `UnexpectedAlertPresentException`!

In this article, we will learn how to shift our driver's focus to these native dialogs using `switch_to.alert`.

---

## 1. The Three Types of JavaScript Dialogs

Before we write code, we must understand the three types of native dialogs:

1. **Alert (`window.alert`)**: Contains a message and a single "OK" button.
2. **Confirm (`window.confirm`)**: Contains a message and two buttons: "OK" and "Cancel".
3. **Prompt (`window.prompt`)**: Contains a message, an input field to type text, and two buttons.

Let's see how to handle all three using Selenium Python.

---

## 2. Handling a Simple Alert

When an alert appears, we must first tell our `WebDriver` to switch its context from the webpage to the active alert. We then read the text and accept it.

```python
from selenium import webdriver
from selenium.webdriver.common.by import By
def test_simple_alert():
    driver = webdriver.Chrome()
    driver.get("https://mycodeyatra.com/alerts")
    # Trigger the alert via the UI
    driver.find_element(By.ID, "trigger-alert").click()
    # 1. Switch focus to the Alert
    alert = driver.switch_to.alert
    # 2. Read the text inside the alert
    print(f"Alert Text: {alert.text}")
    assert alert.text == "This is a simple alert message!"
    # 3. Accept the alert (Clicks 'OK')
    alert.accept()
    driver.quit()
```

---

## 3. Handling a Confirmation Dialog

A Confirmation Dialog gives the user a choice. We can either accept it (OK) or dismiss it (Cancel).

```python
def test_confirm_dialog():
    driver = webdriver.Chrome()
    driver.get("https://mycodeyatra.com/alerts")
    # Trigger the confirm dialog
    driver.find_element(By.ID, "trigger-confirm").click()
    alert = driver.switch_to.alert
    assert alert.text == "Are you sure you want to delete this item?"
    # To click "Cancel", we use dismiss()
    alert.dismiss()
    # Let's verify the UI updated properly after cancelling
    result_text = driver.find_element(By.ID, "result-msg").text
    assert result_text == "Delete cancelled."
    driver.quit()
```

---

## 4. Handling a Prompt Dialog

A Prompt Dialog asks the user for input. We can use the `send_keys()` method directly on the Alert object before accepting it!

```python
def test_prompt_dialog():
    driver = webdriver.Chrome()
    driver.get("https://mycodeyatra.com/alerts")
    # Trigger the prompt
    driver.find_element(By.ID, "trigger-prompt").click()
    alert = driver.switch_to.alert
    # Type our name into the prompt!
    alert.send_keys("Pankaj Kumar")
    # Click OK
    alert.accept()
    # Verify the UI reflects our input
    greeting = driver.find_element(By.ID, "greeting-msg").text
    assert greeting == "Hello, Pankaj Kumar!"
    driver.quit()
```

---

## 5. Dealing with Unexpected Alerts (Explicit Waits)

Sometimes, an alert takes a second to appear (e.g., after an API call finishes). If you call `driver.switch_to.alert` before the alert is actually rendered, Selenium will throw a `NoAlertPresentException`.

To prevent this, we should always use an **Explicit Wait** to wait for the alert to be present!

```python
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
def test_wait_for_alert():
    driver = webdriver.Chrome()
    driver.get("https://mycodeyatra.com/alerts")
    driver.find_element(By.ID, "delayed-alert-btn").click()
    # Wait up to 5 seconds for the alert to appear
    wait = WebDriverWait(driver, 5)
    # EC.alert_is_present() automatically switches to the alert and returns it!
    alert = wait.until(EC.alert_is_present())
    print(f"Successfully caught delayed alert: {alert.text}")
    alert.accept()
    driver.quit()
```

---

## 6. Execution Output

When we run these tests via PyTest, we successfully interact with all four variations seamlessly.

```text
============================= test session starts ==============================
platform win32 -- Python 3.12.0, pytest-8.0.0
rootdir: C:\Automation\mycodeyatra-tests
collected 4 items
test_alerts.py::test_simple_alert PASSED                                 [ 25%]
test_alerts.py::test_confirm_dialog PASSED                               [ 50%]
test_alerts.py::test_prompt_dialog PASSED                                [ 75%]
test_alerts.py::test_wait_for_alert PASSED                               [100%]
============================== 4 passed in 10.45s ==============================
```

## Conclusion

Handling native browser alerts is fundamentally different from handling standard web elements. 
- You cannot inspect them.
- You must shift your driver context using `switch_to.alert`.
- You use `accept()`, `dismiss()`, and `send_keys()` to interact with them.
- You should always use `WebDriverWait` with `alert_is_present()` if the alert generation is asynchronous.

Now that we have conquered popups, we will move on to dealing with the OS layer in our next article: **File Uploads!**
