---
title: Advanced Session Handling: Reusing WebDriver Sessions in Python
date: 25-Feb-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [python, selenium, session, webdriver, performance, reuse]
category: Authentication
categories: [Authentication, Security, Python]
excerpt: >-
  Stop launching new browsers for every test! Learn how to extract the WebDriver Session ID and Executor URL to hijack and reuse existing browser windows, drastically speeding up your automation suite.
readTime: 6 min read
---

# Advanced Session Handling: Reusing WebDriver Sessions in Python

Have you ever noticed how long it takes for a new browser window to launch?

When you call `webdriver.Chrome()`, Selenium has to spin up a new ChromeDriver process, establish a connection, and physically launch the Chrome UI. This takes roughly 2-3 seconds. If you have 500 tests, launching and closing the browser for every single test wastes **25 minutes** of pure overhead!

In this article, we will learn how to extract the WebDriver `session_id` and reuse the exact same browser window across multiple Python scripts without relaunching it.

---

## 1. How WebDriver Sessions Work

When `webdriver.Chrome()` is called, two critical things happen:
1. **The Executor URL:** Selenium starts a local web server (e.g., `http://localhost:52134`) to communicate with the browser driver.
2. **The Session ID:** The driver opens a browser and assigns it a unique hexadecimal Session ID (e.g., `f5e1...`).

To hijack and reuse an existing browser, we simply need to capture these two strings!

---

## 2. Launching and Extracting the Session

Let's write a script that launches the browser, navigates to our site, and then prints out the exact Executor URL and Session ID.

**launch_browser.py**

```python
import time
from selenium import webdriver
def launch_and_hold_browser():
    # 1. Launch a new browser normally
    driver = webdriver.Chrome()
    driver.get("https://mycodeyatra.com")
    # 2. Extract the Executor URL and Session ID
    executor_url = driver.command_executor._url
    session_id = driver.session_id
    print(f"\n[Browser Launched]")
    print(f"Executor URL: {executor_url}")
    print(f"Session ID:   {session_id}")
    print("\nThe browser is now open. Run `hijack_session.py` to control it!")
    # Keep the script alive so the browser doesn't close
    time.sleep(300)
if __name__ == "__main__":
    launch_and_hold_browser()
```

---

## 3. Hijacking the Existing Browser

Now, we will write a totally separate Python script. Instead of calling `webdriver.Chrome()`, we will use `webdriver.Remote()`, pass in the Executor URL we just generated, and explicitly inject our Session ID!

**hijack_session.py**

```python
from selenium import webdriver
def reuse_existing_session(executor_url, session_id):
    # 1. Create a "Remote" WebDriver pointing to the existing ChromeDriver server
    options = webdriver.ChromeOptions()
    driver = webdriver.Remote(command_executor=executor_url, options=options)
    # BUG FIX: Selenium creates a new session by default when calling Remote().
    # We must explicitly overwrite the new session ID with our existing one!
    driver.close() # Close the accidentally opened second window
    driver.session_id = session_id # Inject the hijacked Session ID
    print(f"\n[Hijacked!] Successfully connected to Session: {driver.session_id}")
    # 2. Control the existing browser!
    print(f"Current Title: {driver.title}")
    # Navigate the hijacked browser to a new page
    driver.get("https://github.com")
    print(f"Navigated existing browser to: {driver.current_url}")
if __name__ == "__main__":
    # Paste the values from the first script here:
    URL = "http://localhost:12345"
    SESSION = "abcdef1234567890"
    reuse_existing_session(URL, SESSION)
```

---

## 4. Execution Output

If you run the first script, it will print the URL and Session ID and hold the browser open. If you paste those values into the second script and run it, **it will instantly control the exact same browser window!**

```bash
python launch_browser.py
```
```text
[Browser Launched]
Executor URL: http://localhost:59123
Session ID:   4b2c1f9a8e7d6b5a4c3d2e1f0a9b8c7d
The browser is now open. Run `hijack_session.py` to control it!
```

```bash
python hijack_session.py
```
```text
[Hijacked!] Successfully connected to Session: 4b2c1f9a8e7d6b5a4c3d2e1f0a9b8c7d
Current Title: MyCodeYatra - Learn Test Automation
Navigated existing browser to: https://github.com/
```

## Conclusion

Reusing WebDriver sessions is an advanced optimization technique.
- It completely eliminates browser startup overhead, saving minutes (or hours) across large test suites.
- It allows you to maintain authentication state perfectly without re-logging in.
- You simply need the `driver.command_executor._url` and the `driver.session_id`.
- Use `webdriver.Remote()` to connect to the executor, close the dummy window it spawns, and overwrite the `session_id` property!

In our next article, we will tackle **Multi-User Testing**. How do you test a scenario where an Admin user and a Customer user need to interact with the same system simultaneously?
