---
title: Handling Multiple Windows and Tabs in Selenium Python
date: 12-Dec-2024
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [selenium, python, windows, tabs, window-handles, switch-to]
category: UI Automation
categories: [UI Automation, Selenium, Python]
excerpt: >-
  Stop your tests from crashing when a new tab opens! Learn how to use window_handles to switch driver context across multiple tabs and windows effortlessly.
readTime: 5 min read
---

# Handling Multiple Windows and Tabs in Selenium Python

Have you ever clicked a link on a webpage, only for it to open in a completely new browser tab?

For a human, this is trivial. You just move your eyes to the new tab. But for Selenium, this is a major roadblock. 

Selenium operates inside a single **Context** at a time. If a link opens a new tab, Selenium’s focus *remains on the original tab*. If you try to find an element that only exists on the new tab, Selenium will throw a `NoSuchElementException`.

In this article, we will learn how to detect new windows and switch our driver's context between them using **Window Handles**.

---

## 1. What is a Window Handle?

Every single tab or window opened in a Selenium session is assigned a unique alphanumeric ID called a **Window Handle** (e.g., `CDwindow-1A2B3C`). 

Selenium keeps track of two things:
1. `driver.current_window_handle`: The ID of the tab Selenium is currently focused on.
2. `driver.window_handles`: A list containing the IDs of *all* open tabs.

Let's see how we use these lists to switch context.

---

## 2. Switching to a New Tab

The workflow for switching to a new tab is always the same:
1. Save the ID of the original tab.
2. Click the link that opens the new tab.
3. Iterate through `driver.window_handles`.
4. If a handle is *not* the original tab, switch to it!

```python
import time
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
def test_switch_to_new_tab():
    driver = webdriver.Chrome()
    driver.get("https://mycodeyatra.com/external-links")
    # 1. Save the original tab's ID
    original_window = driver.current_window_handle
    # 2. Click a link that has target="_blank" (opens in new tab)
    driver.find_element(By.ID, "open-partner-site").click()
    # Wait for the new tab to actually open
    WebDriverWait(driver, 10).until(EC.number_of_windows_to_be(2))
    # 3. Loop through all open windows
    for window_handle in driver.window_handles:
        if window_handle != original_window:
            # 4. Switch to the new tab!
            driver.switch_to.window(window_handle)
            break
    # Now we are on the new tab. Let's verify!
    print(f"Current Title: {driver.title}")
    assert "Partner Site" in driver.title
    driver.quit()
```

---

## 3. Switching Back to the Original Tab

When you are done interacting with the new tab, you should always close it and return focus to the original tab so your test can continue.

If you don't close the new tabs, they will eat up your computer's RAM, especially if you are running a massive test suite!

```python
def test_closing_tabs():
    driver = webdriver.Chrome()
    driver.get("https://mycodeyatra.com")
    original_window = driver.current_window_handle
    # Open a new tab using Javascript (Selenium 4 also has driver.switch_to.new_window('tab'))
    driver.switch_to.new_window('tab')
    driver.get("https://google.com")
    assert "Google" in driver.title
    # 1. Close the current (new) tab
    # IMPORTANT: driver.close() closes the current tab. driver.quit() kills the whole browser.
    driver.close()
    # 2. Switch focus back to the original tab
    # If you forget this step, Selenium will be focused on a tab that no longer exists, and will crash!
    driver.switch_to.window(original_window)
    assert "MyCodeYatra" in driver.title
    driver.quit()
```

---

## 4. Handling 3 or More Tabs

If your application opens multiple tabs, the simple `if window_handle != original_window` trick won't work anymore. You won't know *which* of the new tabs is the one you want!

In this scenario, you must iterate through all tabs, switch to them one by one, and check the `driver.title` or `driver.current_url` until you find the right one.

```python
def test_multiple_tabs():
    driver = webdriver.Chrome()
    driver.get("https://mycodeyatra.com/dashboard")
    # Click two different links that open new tabs
    driver.find_element(By.ID, "open-reports").click()
    driver.find_element(By.ID, "open-settings").click()
    # We now have 3 tabs open! Let's find the "Settings" tab.
    for handle in driver.window_handles:
        driver.switch_to.window(handle)
        # If the title matches what we are looking for, stop searching!
        if "Settings" in driver.title:
            break
    # We are now safely focused on the Settings tab
    assert driver.find_element(By.ID, "theme-toggle").is_displayed()
    driver.quit()
```

---

## 5. Execution Output

When we execute these context-switching tests in PyTest, Selenium perfectly hops between the tabs, validates the content, and closes them gracefully.

```text
============================= test session starts ==============================
platform win32 -- Python 3.12.0, pytest-8.0.0
rootdir: C:\Automation\mycodeyatra-tests
collected 3 items
test_windows.py::test_switch_to_new_tab PASSED                           [ 33%]
test_windows.py::test_closing_tabs PASSED                                [ 66%]
test_windows.py::test_multiple_tabs PASSED                               [100%]
============================== 3 passed in 11.40s ==============================
```

## Conclusion

Handling multiple tabs is a necessary skill for complex end-to-end flows (like clicking an email verification link that opens a new tab). 
Remember the golden rule: **Whenever you `driver.close()` a tab, you must manually `switch_to.window()` to give Selenium a new context.**

Now that we understand how to switch contexts between full browser tabs, we will look at how to switch contexts *inside* the same page using **Frames & iFrames**. See you in the next article!
