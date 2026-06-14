---
title: Applitools Eyes: AI-Powered Visual Validation in Python
date: 09-Apr-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [python, selenium, visual-testing, applitools, ai, machine-learning]
category: Visual Testing
categories: [Visual Testing, Python, UI Automation]
excerpt: >-
  Eliminate false positives in Visual Testing! Learn how to integrate Applitools Eyes into your Pytest Selenium framework to replace strict pixel math with Machine Learning Cognitive Vision.
readTime: 6 min read
---

# Applitools Eyes: AI-Powered Visual Validation in Python

In our previous articles, we implemented visual testing using strict pixel matching. While pixel matching is fantastic for small projects, it faces a massive problem at the Enterprise scale: **False Positives.**

If Chrome updates its font-rendering engine, or if an operating system applies a 1-pixel anti-aliasing shadow to your text, a pixel-matching algorithm will declare the test a failure! You will spend hours manually reviewing hundreds of "failed" tests only to realize the application is perfectly fine.

To solve this, the industry standard is **Applitools Eyes**. Instead of comparing pixels, Applitools uses **Cognitive Vision (Machine Learning)**. It looks at the application exactly like a human does. It ignores 1-pixel rendering shifts, but immediately flags missing buttons, broken CSS layouts, or overlapping text!

In this article, we will integrate the official Applitools Python SDK into our Selenium framework.

---

## 1. Installing the Applitools SDK

First, you need to create a free account at [Applitools.com](https://applitools.com) to get your API Key.

Then, install the official Python SDK:

```bash
pip install eyes-selenium
```

---

## 2. Integrating Applitools with Pytest

Applitools makes visual testing incredibly easy. Instead of manually downloading baselines from AWS S3, capturing screenshots, and writing math algorithms, Applitools handles everything automatically in the cloud!

All you have to do is tell Applitools to "open its eyes" at the start of the test, "check the window", and "close its eyes" at the end!

**tests/test_applitools_visual.py**

```python
import os
import pytest
from selenium import webdriver
from applitools.selenium import Eyes, Target
# Define our API Key from the Environment Variable
APPLITOOLS_API_KEY = os.getenv("APPLITOOLS_API_KEY", "YOUR_FREE_API_KEY")
@pytest.fixture(scope="function")
def eyes():
    """Setup Applitools Eyes as a Pytest Fixture"""
    eyes_instance = Eyes()
    eyes_instance.api_key = APPLITOOLS_API_KEY
    yield eyes_instance
    # After the test, ensure eyes are safely closed and test results are finalized
    eyes_instance.abort_if_not_closed()
def test_homepage_ai_visual_regression(eyes):
    driver = webdriver.Chrome()
    driver.set_window_size(1280, 800)
    # 1. Start the Visual Test!
    # Parameters: (driver, App Name, Test Name)
    print("\n[Visual AI] Opening Applitools Eyes...")
    eyes.open(driver, "MyCodeYatra App", "Homepage Desktop View")
    # 2. Navigate the application
    driver.get("https://httpbin.org/")
    # 3. Capture the screen and send it to the Applitools AI Brain
    print("[Visual AI] Checking the main window...")
    eyes.check("Main Dashboard", Target.window().fully())
    # 4. Close the eyes and wait for the AI result!
    # If the AI detects a regression, eyes.close() will throw a DiffsFoundError
    print("[Visual AI] Closing eyes and waiting for cloud analysis...")
    result = eyes.close(raise_ex=True)
    print(f"✅ Applitools Test Passed! View dashboard: {result.url}")
    driver.quit()
```

---

## 3. The Power of Match Levels

By default, Applitools uses the `STRICT` match level, which compares layout and text. 

But what if your homepage has a dynamic advertisement banner that changes every time the page loads? Pixel matching would fail immediately. With Applitools, we can change the AI's Match Level to **LAYOUT** mode!

In `LAYOUT` mode, the AI ignores the content of the images or the text, and *only* verifies that the CSS grid architecture remains intact!

```python
from applitools.selenium import MatchLevel
def test_dynamic_homepage_layout(eyes):
    driver = webdriver.Chrome()
    eyes.open(driver, "MyCodeYatra App", "Dynamic Ads Page")
    # Set the AI to ignore content changes and only verify CSS layout!
    eyes.match_level = MatchLevel.LAYOUT
    driver.get("https://httpbin.org/dynamic_ads")
    eyes.check("Ad Dashboard", Target.window().fully())
    eyes.close()
    driver.quit()
```

---

## 4. Execution Output

Let's execute the Pytest suite. Applitools will take the screenshot and securely stream it to their cloud ML engine.

```bash
pytest test_applitools_visual.py -s
```

```text
============================= test session starts ==============================
platform win32 -- Python 3.12.0, pytest-8.0.0
rootdir: C:\Automation\mycodeyatra-visual
collected 1 item
test_applitools_visual.py 
[Visual AI] Opening Applitools Eyes...
[Visual AI] Checking the main window...
[Visual AI] Closing eyes and waiting for cloud analysis...
✅ Applitools Test Passed! View dashboard: https://eyes.applitools.com/app/test-results/12345
.
============================== 1 passed in 4.90s ===============================
```

## Conclusion

If your company has the budget, Applitools is the ultimate visual testing solution.
- It completely eliminates false positives caused by font rendering or OS anti-aliasing.
- It natively handles dynamic content using `MatchLevel.LAYOUT`.
- It completely eliminates the need for you to manage baseline images in AWS S3 or Git—Applitools stores them all in a beautiful cloud dashboard!

In our next article, we will move away from desktop monitors and explore **Responsive Testing**. How do we visually test our application to ensure it looks perfect on an iPhone, an iPad, and a massive 4K Desktop simultaneously?
