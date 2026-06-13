---
title: Screenshots and Video Recording in Selenium Python
date: 26-Nov-2024
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [selenium, python, pytest, screenshots, video, debugging]
category: UI Automation
categories: [UI Automation, Selenium, Python]
excerpt: >-
  Never wonder why a headless test failed again. Learn how to automatically capture screenshots using PyTest hooks and explore strategies for recording video of your test execution.
readTime: 5 min read
---

# Screenshots and Video Recording in Selenium Python

When your automated tests run on your local machine, you can watch the browser open, navigate, and click. If something fails, you see it happen with your own eyes.

But what happens when you run your tests in a Headless CI/CD server like GitHub Actions or Jenkins? If a test fails, you only get a console error message. If the error says "Element not found", was the element genuinely missing, or did the page fail to load entirely?

To debug headless test failures, we must configure our framework to automatically capture visual evidence: **Screenshots and Videos**.

---

## 1. Taking Basic Screenshots

Taking a screenshot in Selenium Python is incredibly simple thanks to the built-in W3C command `save_screenshot()`.

```python
from selenium import webdriver
def test_manual_screenshot():
    driver = webdriver.Chrome()
    driver.get("https://mycodeyatra.com")
    # 1. Take a full viewport screenshot
    driver.save_screenshot("homepage_viewport.png")
    # 2. Take an element-specific screenshot!
    login_btn = driver.find_element(By.CSS_SELECTOR, "a[href='/login']")
    login_btn.screenshot("login_button_only.png")
    driver.quit()
```

While manually taking screenshots is useful for visually validating components, we usually only care about screenshots **when a test fails**.

---

## 2. Capturing Screenshots Automatically on Failure

We do not want to pollute our test methods with `try-except` blocks just to take screenshots. Instead, we can use a **PyTest Hook**.

PyTest provides a hook called `pytest_runtest_makereport` which executes after every test phase (setup, call, teardown). We can intercept this hook to take a screenshot if the test fails.

Create a file named `conftest.py` in your project root:

```python
import pytest
from selenium import webdriver
# A fixture to initialize the driver and make it available to the test
@pytest.fixture
def driver(request):
    driver = webdriver.Chrome()
    request.node.driver = driver  # Attach driver to the test node
    yield driver
    driver.quit()
# PyTest Hook that runs after every test executes
@pytest.hookimpl(tryfirst=True, hookwrapper=True)
def pytest_runtest_makereport(item, call):
    # Execute all other hooks to obtain the report object
    outcome = yield
    report = outcome.get_result()
    # We only care about the actual test execution ("call" phase), not setup/teardown
    if report.when == "call" and report.failed:
        # Check if the driver was attached to the node
        driver = getattr(item, "driver", None)
        if driver is not None:
            screenshot_path = f"screenshots/{item.name}_failed.png"
            driver.save_screenshot(screenshot_path)
            print(f"\n[FAILURE] Screenshot saved to: {screenshot_path}")
```

Now, your test logic remains incredibly clean!

```python
def test_failing_login(driver):
    driver.get("https://mycodeyatra.com/login")
    # This element does not exist, so the test will fail
    fake_element = driver.find_element(By.ID, "does-not-exist")
```
When `test_failing_login` inevitably crashes, PyTest will instantly trigger our hook, grab the `driver` object, and save `test_failing_login_failed.png` to our screenshots folder!

---

## 3. Video Recording the Test Execution

Screenshots show you the exact moment of failure, but sometimes you need to see the *lead-up* to the failure. Was the page flickering? Was a modal animating?

Selenium WebDriver does not have built-in video recording capabilities. However, Python provides amazing third-party solutions. 

The easiest way to record videos cross-platform is using a screen capture library like `pytest-video` or `mss`. But in modern testing architectures, it is actually much better to use cloud infrastructure like Selenium Grid or Docker containers (like Selenoid/Zalenium) that record the video natively at the OS layer.

If you want to record locally without Docker, you can use the `mss` screen-capture library combined with OpenCV to compile a video frame-by-frame:

```bash
pip install mss opencv-python
```

While writing a custom video recorder in Python is possible, we highly recommend utilizing Dockerized Selenium Grid for enterprise-scale video recording, which we will cover extensively in Phase 5 of our roadmap!

## Conclusion & End of Phase 1

You have done it! You have successfully completed **Phase 1: Foundations** of the Selenium Python curriculum.

You now know how to install Python, manage dependencies, launch drivers natively, locate elements, write robust PyTest assertions, execute End-to-End tests, and capture debugging logs and screenshots. 

However, right now, our test code is still quite messy. In **Phase 2: Framework Architecture**, we will elevate your code to an Enterprise level by implementing the Page Object Model (POM), PyTest Fixtures, Data-Driven Testing, and custom configurations. 

See you in Phase 2!
