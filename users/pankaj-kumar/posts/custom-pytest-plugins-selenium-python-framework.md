---
title: Creating Custom PyTest Plugins for Selenium Frameworks
date: 25-Jan-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [selenium, python, pytest, plugins, framework, enterprise]
category: Framework Architecture
categories: [Framework Architecture, Selenium, Python]
excerpt: >-
  Stop copy-pasting code across repositories! Learn how to package your Selenium fixtures and hooks into a custom PyTest plugin that can be pip installed by any team in your company.
readTime: 6 min read
---

# Creating Custom PyTest Plugins for Selenium Frameworks

Throughout this phase, we built an incredible Enterprise Architecture. We have a robust `DriverFactory`, a `ConfigReader`, and a massive `conftest.py` file filled with global fixtures and hooks.

But what happens when your company starts a *second* automation project for a completely different application?

Do you copy and paste your `conftest.py` and `DriverFactory` into the new repository? **No! Copy-pasting violates the DRY (Don't Repeat Yourself) principle.**

If you have framework code that needs to be shared across multiple repositories, you should package it into a **Custom PyTest Plugin**.

---

## 1. What is a PyTest Plugin?

A PyTest Plugin is simply a standard Python package that exposes fixtures and hooks using the `pytest11` entry point. 

When you `pip install` a PyTest plugin, PyTest automatically discovers its fixtures, making them instantly available to any test running on your machine, without a single import statement!

In fact, `pytest-xdist` and `pytest-html` are just PyTest Plugins built by other developers. We are going to build our own!

---

## 2. Building Your Plugin

Let's imagine we are building a plugin called `pytest-mycodeyatra-selenium`. We want it to automatically provide our standard `driver` fixture to any project that installs it.

Create a completely separate folder (outside of your test framework) with this structure:

```text
pytest-mycodeyatra-selenium/
│
├── pytest_mycodeyatra.py     # The actual plugin code
├── setup.py                  # The pip installer configuration
└── README.md
```

### The Plugin Code
**pytest_mycodeyatra.py**

```python
import pytest
from selenium import webdriver
# 1. Define custom CLI options for our plugin
def pytest_addoption(parser):
    parser.addoption(
        "--mycodeyatra-browser", 
        action="store", 
        default="chrome", 
        help="Browser to run tests on via mycodeyatra plugin"
    )
# 2. Define the globally available fixture
@pytest.fixture
def driver(request):
    """
    Provides a configured Selenium WebDriver instance.
    This will become available to EVERY project that installs this plugin!
    """
    browser = request.config.getoption("--mycodeyatra-browser").lower()
    print(f"\n[Plugin] Initializing {browser} driver...")
    if browser == "chrome":
        driver_instance = webdriver.Chrome()
    elif browser == "firefox":
        driver_instance = webdriver.Firefox()
    else:
        raise ValueError("Unsupported browser")
    driver_instance.maximize_window()
    yield driver_instance
    print(f"\n[Plugin] Tearing down {browser} driver...")
    driver_instance.quit()
```

---

## 3. Configuring the Plugin Entry Point

This is the most critical step. We must configure `setup.py` and define the `pytest11` entry point. This tells PyTest, *"Hey! I have fixtures! Load me automatically!"*

**setup.py**

```python
from setuptools import setup
setup(
    name='pytest-mycodeyatra-selenium',
    version='0.1.0',
    description='A custom PyTest plugin for enterprise Selenium configuration',
    author='MyCodeYatra',
    py_modules=['pytest_mycodeyatra'],
    install_requires=['pytest', 'selenium'],
    # THIS IS THE MAGIC LINE!
    entry_points={
        'pytest11': [
            'mycodeyatra = pytest_mycodeyatra',
        ],
    },
)
```

---

## 4. Installing and Using the Plugin

Now, navigate to the `pytest-mycodeyatra-selenium/` folder in your terminal and install it locally using `pip`:

```bash
pip install -e .
```

Congratulations! Your plugin is now installed globally on your machine.

Now, open a completely blank, brand-new Python project. Do **not** create a `conftest.py` file. Do **not** write a driver factory.

Just create a test file:

**test_plugin.py**

```python
# No imports needed! The 'driver' fixture is provided by our Plugin!
def test_plugin_magic(driver):
    driver.get("https://mycodeyatra.com")
    assert "MyCodeYatra" in driver.title
```

---

## 5. Execution Output

When we run PyTest in this brand-new project, it automatically loads our custom plugin, registers our custom CLI argument `--mycodeyatra-browser`, and injects the `driver` fixture!

```bash
pytest test_plugin.py -s --mycodeyatra-browser=firefox
```

```text
============================= test session starts ==============================
platform win32 -- Python 3.12.0, pytest-8.0.0
plugins: mycodeyatra-selenium-0.1.0
collected 1 item
test_plugin.py 
[Plugin] Initializing firefox driver...
PASSED
[Plugin] Tearing down firefox driver...
============================== 1 passed in 4.95s ===============================
```

## Conclusion & End of Phase 4

By packaging your core framework logic into a Custom PyTest Plugin, you create the ultimate enterprise architecture. Your Central QA Team can maintain the `pytest-company-core` plugin, and dozens of different dev teams can simply `pip install` it to get instant access to standardized driver configurations, reporting hooks, and API tools!

Congratulations! You have completed **Phase 4: Framework Architecture**. 

You now possess the skills to build, scale, and distribute massive Python test automation suites. However, web applications are rarely just UIs. They are powered by complex backend APIs. 

In **Phase 5: API Testing**, we will pivot away from Selenium and learn how to use Python's legendary `Requests` library to automate REST APIs! See you there!
