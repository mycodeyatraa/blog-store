---
title: WebDriver Lifecycle: Initialization, Navigation, and Teardown
date: 13-Nov-2024
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [selenium, python, webdriver, lifecycle, teardown, context-managers]
category: UI Automation
categories: [UI Automation, Selenium, Python]
excerpt: >-
  Master the Selenium WebDriver lifecycle in Python. Learn the critical difference between close() and quit(), and how to use Python Context Managers to prevent catastrophic memory leaks.
readTime: 5 min read
---

# WebDriver Lifecycle: Initialization, Navigation, and Teardown

In our last article, we learned that calling `webdriver.Chrome()` physically boots up a `chromedriver.exe` server in the background and opens a network connection to it.

But what happens when your test finishes? Does that server magically shut down? Does the memory get returned to your OS? 

In this article, we will explore the **WebDriver Lifecycle**. Understanding how to properly initialize, navigate, and destroy browser sessions is the difference between a stable framework and a CI/CD pipeline that crashes every night due to "Out of Memory" errors.

---

## 1. Initialization

Every test begins by initializing a new session. In Python, this is extremely straightforward:

```python
from selenium import webdriver
 
# Starts the chromedriver server and opens a Chrome window
driver = webdriver.Chrome()
 
# Maximize the window to ensure responsive layouts don't hide elements
driver.maximize_window()

```

When this line executes, Python receives a unique **Session ID**. For the rest of this test, every command will be tied to this specific ID.

---

## 2. Navigation Commands

Once the session is active, we can instruct the browser to navigate. The `driver` object exposes several methods to control the browser's history and URL state:

```python
# 1. Navigate to a URL (Blocks execution until the page fully loads)
driver.get("https://mycodeyatra.com/login")
 
# 2. Refresh the current page
driver.refresh()
 
# 3. Navigate to a second URL
driver.get("https://mycodeyatra.com/dashboard")
 
# 4. Press the Browser's "Back" button (Returns to /login)
driver.back()
 
# 5. Press the Browser's "Forward" button (Returns to /dashboard)
driver.forward()

```

*(Note: `driver.get()` waits for the JavaScript `document.readyState` to equal `"complete"`. If a webpage has a lot of heavy tracking scripts or ads, this command might hang. We will discuss advanced loading strategies in future articles).*

---

## 3. Teardown: `close()` vs `quit()`

This is the most common mistake made by Junior Automation Engineers. What is the difference between these two commands?

### `driver.close()`
This command tells the browser to close the **current active tab or window**. 
If you have 3 tabs open, calling `.close()` closes only the one you are currently focused on. If you close the very last tab, the browser application closes—but **the `chromedriver.exe` server remains running in the background!**

### `driver.quit()`
This is the command you should use 99% of the time. `.quit()` sends a `DELETE /session/{id}` request to the driver. It closes **all tabs**, closes the browser application, and gracefully **kills the `chromedriver.exe` background process**, freeing up your machine's RAM.

### The Orphaned Process Problem
If your test throws an Exception and fails *before* it reaches the `driver.quit()` line, the background process will stay alive forever. If you run 500 tests, you will eventually have 500 hidden Chrome instances eating all of your RAM until your computer completely freezes!

We will solve this later using `pytest` teardown fixtures, but for now, always ensure `.quit()` is called.

---

## 4. The Pythonic Way: Context Managers

Python has a beautiful feature called Context Managers (`with` blocks) that guarantees teardown code executes, even if an exception occurs!

Because the Selenium WebDriver natively implements the `__enter__` and `__exit__` context manager methods, you can write bulletproof lifecycle code like this:

```python
import time
from selenium import webdriver
from selenium.webdriver.common.by import By
 
def test_login_lifecycle():
 
    # The 'with' block initializes the driver
    with webdriver.Chrome() as driver:
        driver.maximize_window()
        driver.get("https://mycodeyatra.com")
 
        # If this line fails (Element Not Found Exception), the test stops...
        driver.find_element(By.ID, "non-existent-button").click()
 
    # ...But because we used 'with', Python GUARANTEES that driver.quit() 
    # is called automatically the second we exit the block!
    print("Browser safely destroyed, even after failure!")

```

## Conclusion

Understanding the lifecycle of the WebDriver ensures your automation framework is a good citizen on your machine, cleaning up its own resources.

However, navigating to URLs is only step one. To actually test a web application, we need to interact with buttons, inputs, and text. In our next article, we will dive into the most fundamental skill in all of UI automation: **Finding elements using Locators!**
