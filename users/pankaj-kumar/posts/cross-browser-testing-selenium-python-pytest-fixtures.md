---
title: Cross-Browser Testing in Selenium Python
date: 25-Dec-2024
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [selenium, python, cross-browser, pytest, fixtures, selenium-manager]
category: UI Automation
categories: [UI Automation, Selenium, Python]
excerpt: >-
  Don't just test in Chrome! Learn how to use parameterized PyTest fixtures to seamlessly execute your single test suite across Chrome, Firefox, and Edge simultaneously.
readTime: 5 min read
---

# Cross-Browser Testing in Selenium Python

In the early days of the internet, a website that looked perfect in Chrome might be completely broken in Internet Explorer. Today, browsers are much more standardized, but rendering engines (Blink for Chrome/Edge, Gecko for Firefox, WebKit for Safari) still have subtle differences that can break your application.

Your users are diverse. Some use Chrome on Windows, others use Safari on Mac. If you only test your application in Chrome, you are blind to a huge portion of your user base.

In this final article of Phase 2, we will explore how to write a single test suite and execute it across multiple browsers using **Cross-Browser Testing**.

---

## 1. The Power of Selenium Manager

Historically, if you wanted to run a test on Firefox, you had to manually download `geckodriver.exe`. For Edge, `msedgedriver.exe`. Managing these driver binaries across a team was a nightmare.

Thankfully, Selenium 4.6+ introduced **Selenium Manager**. It automatically downloads and manages the driver binaries for you in the background.

Launching different browsers is now as simple as calling a different class:

```python
from selenium import webdriver
# Launch Google Chrome
driver_chrome = webdriver.Chrome()
# Launch Mozilla Firefox
driver_firefox = webdriver.Firefox()
# Launch Microsoft Edge
driver_edge = webdriver.Edge()
# Launch Apple Safari (Mac ONLY)
# driver_safari = webdriver.Safari()
```

---

## 2. Implementing Cross-Browser Testing in PyTest

We don't want to duplicate our test files (e.g., `test_login_chrome.py` and `test_login_firefox.py`). We want to write the test *once* and tell PyTest to run it against multiple browsers dynamically.

To do this, we use PyTest's **Parametrized Fixtures**.

Create a `conftest.py` file to hold our driver configuration:

```python
import pytest
from selenium import webdriver
# We parameterize the fixture with the names of the browsers we want to test
@pytest.fixture(params=["chrome", "firefox", "edge"])
def driver(request):
    browser = request.param
    if browser == "chrome":
        driver_instance = webdriver.Chrome()
    elif browser == "firefox":
        driver_instance = webdriver.Firefox()
    elif browser == "edge":
        driver_instance = webdriver.Edge()
    else:
        raise ValueError(f"Unsupported browser: {browser}")
    driver_instance.maximize_window()
    # Yield the driver to the test
    yield driver_instance
    # Teardown
    driver_instance.quit()
```

Now, let's write our actual test in `test_search.py`:

```python
from selenium.webdriver.common.by import By
def test_homepage_search(driver):
    driver.get("https://mycodeyatra.com")
    search_box = driver.find_element(By.NAME, "q")
    search_box.send_keys("Selenium Python")
    search_box.submit()
    assert "Selenium Python" in driver.title
```

### What happens when we run this?
Because the `driver` fixture is parameterized with 3 values, PyTest will automatically run `test_homepage_search` **three separate times**—once for Chrome, once for Firefox, and once for Edge!

---

## 3. Combining Cross-Browser with Parallel Execution

Running tests across 3 browsers means your test suite now takes 3x longer to run. 

But wait! In our last article, we learned about `pytest-xdist`. Can we combine Cross-Browser Testing with Parallel Execution? **Yes, we absolutely can.**

If we run the following command in our terminal:

```bash
pytest test_search.py -n 3
```

`pytest-xdist` will spawn Chrome, Firefox, and Edge *at the exact same time*! You will physically see all three different browsers pop open on your screen, execute the test simultaneously, and close.

---

## 4. Execution Output

Let's look at the PyTest console output when we run our parameterized cross-browser test suite. Notice how PyTest automatically appends the parameter name (`[chrome]`, `[firefox]`, `[edge]`) to the test result!

```text
============================= test session starts ==============================
platform win32 -- Python 3.12.0, pytest-8.0.0
plugins: xdist-3.5.0
gw0 [3] / gw1 [3] / gw2 [3]
test_search.py::test_homepage_search[chrome] PASSED                      [ 33%]
test_search.py::test_homepage_search[firefox] PASSED                     [ 66%]
test_search.py::test_homepage_search[edge] PASSED                        [100%]
============================== 3 passed in 14.88s ==============================
```

## Conclusion & End of Phase 2

Congratulations! You have officially completed **Phase 2: Core UI Automation** of the Selenium Python curriculum.

You now possess the skills to handle complex waits, manipulate nested iframes, interact with shadow DOMs, mock API requests, and execute massively scalable cross-browser test suites in parallel.

However, our actual test code structure is still quite basic. If the HTML of our login page changes, we have to update 50 different test files! 

In **Phase 3: Framework Architecture**, we will solve this maintenance nightmare by designing an industry-standard **Page Object Model (POM)** framework. See you in Phase 3!
