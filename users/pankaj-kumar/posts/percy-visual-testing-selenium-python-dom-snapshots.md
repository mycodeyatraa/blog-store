---
title: Percy Visual Testing: Cloud-Scale DOM Snapshots with Python
date: 19-Apr-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [python, selenium, visual-testing, percy, browserstack, cross-browser]
category: Visual Testing
categories: [Visual Testing, Python, UI Automation]
excerpt: >-
  Stop taking flat PNG screenshots! Learn how to use the Percy Python SDK to capture live HTML/CSS DOM snapshots and render them across dozens of cross-browser viewports simultaneously in the BrowserStack cloud.
readTime: 6 min read
---

# Percy Visual Testing: Cloud-Scale DOM Snapshots with Python

In our previous visual testing articles, we relied on capturing `.png` screenshots. Whether we used `pixelmatch` or Applitools, the fundamental architecture involved taking a picture of the browser and analyzing that picture.

**BrowserStack's Percy** flips this architecture entirely on its head.

When you use Percy, your Selenium script does *not* take a screenshot. Instead, Percy captures the complete DOM tree (HTML) and all associated assets (CSS, Images, Fonts). It uploads that raw code to the Percy Cloud, where Percy's massive server farm renders the page across dozens of different browsers (Chrome, Firefox, Safari) and screen sizes simultaneously!

In this final article of Phase 8, we will learn how to integrate the official Percy Python SDK to achieve massive cloud-scale visual testing!

---

## 1. Installing the Percy Architecture

Percy requires two components:
1. The Percy CLI (Command Line Interface), which intercepts the DOM snapshots and uploads them to the cloud.
2. The Python SDK, which tells Selenium *when* to take the snapshot.

First, install the Python SDK:

```bash
pip install percy-selenium
```

Next, you must have Node.js installed to run the Percy CLI. Install the CLI globally:

```bash
npm install -g @percy/cli
```

Finally, grab your `PERCY_TOKEN` from your Percy dashboard and set it as an environment variable!

---

## 2. Integrating Percy with Pytest

Integrating Percy into your Pytest framework is incredibly easy. All you have to do is import `percy_snapshot` and pass it your Selenium WebDriver instance!

**tests/test_percy_visuals.py**

```python
import os
import pytest
from selenium import webdriver
from percy import percy_snapshot
@pytest.fixture(scope="function")
def driver():
    # Setup Chrome
    options = webdriver.ChromeOptions()
    options.add_argument("--headless=new")
    driver_instance = webdriver.Chrome(options=options)
    yield driver_instance
    driver_instance.quit()
def test_homepage_percy_snapshot(driver):
    # 1. Navigate to the application normally
    print("\n[Selenium] Navigating to the target application...")
    driver.get("https://httpbin.org/")
    # 2. Capture the DOM Snapshot!
    # Percy does NOT take a screenshot here. It captures the raw HTML/CSS!
    print("[Percy] Capturing DOM state and sending to BrowserStack Cloud...")
    percy_snapshot(driver, "Homepage Visual Baseline")
    print("✅ Snapshot captured! Review the visual diffs in the Percy Dashboard.")
```

---

## 3. Executing the Percy CLI

Because Percy captures the DOM and uploads it asynchronously, you do not run `pytest` directly. Instead, you wrap your test execution inside the `percy exec` command!

The Percy CLI will boot up, listen for snapshots from Python, and batch upload them to the cloud!

```bash
# Ensure your token is set!
export PERCY_TOKEN="your_secret_token_here"
# Wrap Pytest in the Percy Execution environment
percy exec -- pytest test_percy_visuals.py -s
```

```text
[percy] Percy has started!
[percy] Created build #1: https://percy.io/mycodeyatra/app/builds/12345
============================= test session starts ==============================
platform win32 -- Python 3.12.0, pytest-8.0.0
rootdir: C:\Automation\mycodeyatra-visual
collected 1 item
test_percy_visuals.py 
[Selenium] Navigating to the target application...
[Percy] Capturing DOM state and sending to BrowserStack Cloud...
✅ Snapshot captured! Review the visual diffs in the Percy Dashboard.
.
============================== 1 passed in 3.12s ===============================
[percy] Stopping percy...
[percy] Waiting for 1 snapshot(s) to finish uploading
[percy] Uploaded 1 snapshot(s)
[percy] Finalized build #1: https://percy.io/mycodeyatra/app/builds/12345
```

---

## 4. The Power of Cross-Browser Rendering

The true magic of Percy happens in the cloud. 

If you configure your Percy project to render across Chrome, Firefox, and Safari at 3 different screen sizes (Mobile, Tablet, Desktop), the `percy_snapshot()` command will result in **9 different visual comparisons** in the cloud!

You only ran the Selenium test *once* in Chrome, but Percy rendered the DOM in Safari and Firefox on its own servers! This eliminates the need to run massive, slow cross-browser test suites locally!

## Conclusion

Percy fundamentally changes how we approach visual testing.
- Instead of downloading flat `.png` images from AWS S3, Percy captures the underlying HTML/CSS structure.
- Instead of running Selenium tests on Safari (which is notoriously difficult), you simply send the DOM to Percy, and it renders it in Safari for you!
- Always use the `percy exec` wrapper to ensure your Pytest suite can communicate with the Percy CLI daemon.

This concludes **Phase 8: Visual Testing!** 

You now have a complete arsenal of UI validation tools: from strict `pixelmatch` math, to Applitools' Machine Learning AI, to Percy's Cloud DOM rendering!

In our next segment, **Phase 9: CI/CD Pipelines**, we will pull our entire framework together. We will teach you how to write GitHub Actions and Jenkins pipelines to automatically execute your Selenium tests on every single Pull Request!
