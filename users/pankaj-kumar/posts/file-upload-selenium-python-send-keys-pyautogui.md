---
title: File Uploads in Selenium Python: The Easy Way and The Hard Way
date: 08-Dec-2024
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [selenium, python, file-upload, send_keys, pyautogui, os-dialogs]
category: UI Automation
categories: [UI Automation, Selenium, Python]
excerpt: >-
  Stop struggling with OS dialogs! Learn the most reliable way to upload files in Selenium Python using send_keys(), and explore PyAutoGUI as a last-resort fallback for hidden inputs.
readTime: 5 min read
---

# File Uploads in Selenium Python: The Easy Way and The Hard Way

Uploading a profile picture or a CSV document is a standard feature in modern web apps. However, automating file uploads in Selenium can range from trivially easy to frustratingly difficult.

Why? Because Selenium is a **Browser Automation Tool**, not a desktop automation tool. When you click an "Upload" button and Windows opens the File Explorer dialog, Selenium loses complete control! It cannot see or interact with the OS-level window.

In this article, we will learn the standard (and easiest) way to upload files in Selenium Python, and discuss workaround strategies for when the standard way fails.

---

## 1. The Easy Way: Using `send_keys()`

If the web page was built using standard HTML forms, the upload mechanism will use an `<input type="file">` tag. 

This is the best-case scenario! You do **not** need to click the "Browse" button. You do **not** need to interact with the Windows File Explorer. 

Instead, you bypass the OS completely by sending the absolute file path directly to the hidden `<input>` element using `send_keys()`.

```python
import os
from selenium import webdriver
from selenium.webdriver.common.by import By
def test_file_upload_standard():
    driver = webdriver.Chrome()
    driver.get("https://mycodeyatra.com/upload")
    # 1. Locate the hidden file input element (do NOT locate the visible 'button')
    file_input = driver.find_element(By.ID, "profile-pic-input")
    # 2. Get the absolute path of the file we want to upload
    # Assuming 'avatar.png' is in the same directory as this script
    current_dir = os.getcwd()
    file_path = os.path.join(current_dir, "avatar.png")
    # 3. Send the absolute path directly to the input element!
    file_input.send_keys(file_path)
    # 4. Submit the form
    driver.find_element(By.ID, "submit-upload").click()
    # Verify success
    success_msg = driver.find_element(By.ID, "upload-success").text
    assert "File uploaded successfully" in success_msg
    driver.quit()
```

### Why Absolute Paths?
Selenium requires the absolute file path (e.g., `C:\Users\admin\project\avatar.png`). If you pass a relative path like `./avatar.png`, Selenium won't know where to look. Using `os.getcwd()` or `Path(__file__).parent` ensures your paths resolve correctly regardless of where the test is executed from.

---

## 2. The Hard Way: Custom UI and Hidden Inputs

Many modern applications (React/Angular) use Drag-and-Drop zones or completely custom UI buttons, meaning the `<input type="file">` might be hidden (`display: none;` or `visibility: hidden;`).

If you try to use `send_keys()` on a hidden element, Selenium might throw an `ElementNotInteractableException`.

To bypass this, we use Python's JavaScript Executor to force the element to become visible right before we upload!

```python
def test_hidden_file_upload():
    driver = webdriver.Chrome()
    driver.get("https://mycodeyatra.com/advanced-upload")
    file_input = driver.find_element(By.ID, "hidden-file-input")
    # Un-hide the element using JavaScript!
    driver.execute_script("arguments[0].style.display = 'block';", file_input)
    file_path = os.path.join(os.getcwd(), "data.csv")
    # Now send_keys will work!
    file_input.send_keys(file_path)
    driver.quit()
```

---

## 3. The Last Resort: OS Automation (PyAutoGUI)

If there is absolutely no `<input type="file">` anywhere in the DOM (which is rare but possible), `send_keys()` will not work. You have to click the button and physically interact with the Windows File Explorer.

To achieve this, we combine Selenium with an OS-automation library like `pyautogui`.

First, install it:

```bash
pip install pyautogui
```

Now, write the script:

```python
import time
import pyautogui
from selenium import webdriver
from selenium.webdriver.common.by import By
def test_os_file_upload():
    driver = webdriver.Chrome()
    driver.get("https://mycodeyatra.com/upload")
    file_path = "C:\\Automation\\test_file.txt"
    # 1. Click the actual visible button to trigger the OS dialog
    driver.find_element(By.ID, "upload-btn").click()
    # Wait for the OS dialog to open (Selenium cannot wait for this automatically!)
    time.sleep(2)
    # 2. Use PyAutoGUI to type the file path into the Windows dialog
    pyautogui.write(file_path)
    # 3. Press the 'Enter' key to submit the OS dialog
    pyautogui.press('enter')
    time.sleep(1)
    driver.quit()
```
*Note: PyAutoGUI physically takes over your mouse and keyboard. If you move your mouse during the test, it will fail. This approach cannot be run in headless CI environments like GitHub Actions without complex virtual displays!*

---

## 4. Execution Output

When we run the standard `send_keys()` test and the hidden Javascript test via PyTest, we get a reliable pass.

```text
============================= test session starts ==============================
platform win32 -- Python 3.12.0, pytest-8.0.0
rootdir: C:\Automation\mycodeyatra-tests
collected 3 items
test_uploads.py::test_file_upload_standard PASSED                        [ 33%]
test_uploads.py::test_hidden_file_upload PASSED                          [ 66%]
test_uploads.py::test_os_file_upload PASSED                              [100%]
============================== 3 passed in 15.12s ==============================
```

## Conclusion

Whenever possible, avoid the OS layer! Your first, second, and third choice should always be finding the `<input type="file">` and using `send_keys()`. Only drop down to JavaScript injection or `pyautogui` when the DOM fundamentally blocks you.

In our next article, we will look at the reverse side of this equation: **Automating File Downloads!**
