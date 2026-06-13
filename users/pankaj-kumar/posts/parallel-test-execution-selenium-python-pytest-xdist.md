---
title: Parallel Test Execution in Selenium Python using pytest-xdist
date: 22-Dec-2024
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [selenium, python, parallel-testing, pytest, pytest-xdist, performance]
category: UI Automation
categories: [UI Automation, Selenium, Python]
excerpt: >-
  Cut your test execution time in half! Learn how to scale your Selenium Python framework by running tests concurrently across multiple CPU cores using the pytest-xdist plugin.
readTime: 5 min read
---

# Parallel Test Execution in Selenium Python using pytest-xdist

If you have 10 UI tests that take 10 seconds each, your test suite will take 100 seconds to run. If you scale up to 500 tests, your suite will take over an hour.

Running UI tests sequentially (one after another) is the biggest bottleneck in continuous integration. Developers hate waiting for builds to turn green.

To fix this, we must run our tests **Concurrently**. In Python, the absolute best way to achieve parallel execution is by using a PyTest plugin called `pytest-xdist`.

---

## 1. Installing pytest-xdist

`pytest-xdist` extends PyTest by adding a new command-line argument that distributes your tests across multiple CPU workers.

Install it via pip:

```bash
pip install pytest-xdist
```

---

## 2. Running Tests in Parallel

Let's assume you have a file named `test_search.py` containing 4 different test methods:
- `test_search_laptop()`
- `test_search_phone()`
- `test_search_monitor()`
- `test_search_keyboard()`

To run these tests sequentially, you would use:

```bash
pytest test_search.py
```

To run them in **parallel**, you simply add the `-n` flag, followed by the number of workers (CPU cores) you want to use:

```bash
pytest test_search.py -n 4
```

When you run this, `pytest-xdist` will instantly spawn 4 separate Chrome browser windows at the exact same time, execute all 4 tests simultaneously, and merge the results back into a single terminal output!

### The "Auto" Flag
If you don't know how many CPU cores your machine has (or if you are running in a CI environment where the node size changes), you can use `auto`.

```bash
pytest test_search.py -n auto
```
This tells `xdist` to count the number of logical cores on the machine and spawn exactly that many workers.

---

## 3. The Golden Rule of Parallel Execution: Test Isolation

Parallel execution is amazing, but it exposes poorly written tests instantly.

If your tests share state, they will fail randomly when run in parallel. This is called a **Race Condition**.

### Bad Example (Shared State):

```python
# WARNING: Do NOT do this!
global_driver = webdriver.Chrome()
def test_login():
    global_driver.get("https://mycodeyatra.com/login")
    # ... login logic ...
def test_dashboard():
    # This test assumes the previous test already logged in!
    global_driver.find_element(By.ID, "settings")
```
If you run these two tests in parallel with `-n 2`, `test_dashboard` might execute *before* `test_login` finishes, causing it to crash immediately because the browser isn't logged in yet!

### Good Example (Total Isolation):
To run tests in parallel, **every single test must be 100% independent**. It must launch its own browser, perform its own setup, execute its logic, and tear down its own browser.

```python
import pytest
from selenium import webdriver
# Use a PyTest Fixture to guarantee a fresh driver for every test
@pytest.fixture
def driver():
    driver = webdriver.Chrome()
    yield driver
    driver.quit()
def test_login(driver):
    driver.get("https://mycodeyatra.com/login")
    # ... login logic ...
def test_dashboard(driver):
    # This test MUST login itself before testing the dashboard
    driver.get("https://mycodeyatra.com/login")
    # ... login logic ...
    driver.find_element(By.ID, "settings").click()
```

By ensuring every test has its own isolated `driver` instance via PyTest fixtures, `xdist` can safely distribute them to different CPU cores without any overlap!

---

## 4. Execution Output: Sequential vs Parallel

Let's look at the time difference when running 4 heavy UI tests.

### Sequential Run (`pytest test_search.py`)

```text
============================= test session starts ==============================
collected 4 items
test_search.py::test_search_laptop PASSED                                [ 25%]
test_search.py::test_search_phone PASSED                                 [ 50%]
test_search.py::test_search_monitor PASSED                               [ 75%]
test_search.py::test_search_keyboard PASSED                              [100%]
============================== 4 passed in 41.22s ==============================
```

### Parallel Run (`pytest test_search.py -n 4`)

```text
============================= test session starts ==============================
plugins: xdist-3.5.0
gw0 [4] / gw1 [4] / gw2 [4] / gw3 [4]
....
============================== 4 passed in 11.50s ==============================
```

By adding two characters (`-n 4`) to our terminal command, we reduced our execution time from 41 seconds down to 11 seconds!

## Conclusion

Parallel execution is the key to scaling your automation framework. 
- Use `pytest-xdist`.
- Run tests concurrently using `pytest -n auto`.
- Ensure strict **Test Isolation** so your tests do not interfere with one another.

In our final article of the Core UI series, we will look at how to run your tests across different browsers simultaneously: **Cross-Browser Testing!**
