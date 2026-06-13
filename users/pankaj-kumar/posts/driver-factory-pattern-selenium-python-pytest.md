---
title: The Driver Factory Pattern in Selenium Python
date: 02-Jan-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [selenium, python, driver-factory, design-pattern, pytest, framework]
category: Framework Architecture
categories: [Framework Architecture, Selenium, Python]
excerpt: >-
  Abstract browser instantiation out of your tests! Learn how to build a scalable Driver Factory to manage Chrome, Firefox, and Headless execution dynamically via PyTest CLI arguments.
readTime: 5 min read
---

# The Driver Factory Pattern in Selenium Python

In the previous article, we fixed our framework's fragility by implementing the Page Object Model (POM). We successfully separated our locators from our test logic.

However, if you look at your `conftest.py` file right now, it probably looks something like this:

```python
@pytest.fixture
def driver():
    # What if we want to run Firefox today? 
    # What if we need headless mode?
    driver = webdriver.Chrome() 
    driver.maximize_window()
    yield driver
    driver.quit()
```

The browser initialization logic is hardcoded inside the PyTest fixture. If you want to run your tests in "Headless" mode for your CI/CD pipeline, you have to manually edit this file!

To build an enterprise framework, we need to abstract the browser initialization out of PyTest and into a dedicated **Driver Factory**.

---

## 1. What is the Factory Pattern?

The Factory Pattern is a Creational Design Pattern used in Object-Oriented Programming. 
Instead of calling a constructor directly (`webdriver.Chrome()`), you call a "Factory" method. You pass the Factory a string (like `"firefox"`), and the Factory returns the fully configured object to you.

Let's build a `driver_factory.py` file inside our framework!

---

## 2. Implementing the Driver Factory

We will create a class called `DriverFactory`. It will have a single static method called `get_driver(browser_name)`.

**core/driver_factory.py**

```python
from selenium import webdriver
from selenium.webdriver.chrome.options import Options as ChromeOptions
from selenium.webdriver.firefox.options import Options as FirefoxOptions
class DriverFactory:
    @staticmethod
    def get_driver(browser_name="chrome", headless=False):
        browser_name = browser_name.lower().strip()
        if browser_name == "chrome":
            options = ChromeOptions()
            if headless:
                options.add_argument("--headless=new")
            driver = webdriver.Chrome(options=options)
        elif browser_name == "firefox":
            options = FirefoxOptions()
            if headless:
                options.add_argument("-headless")
            driver = webdriver.Firefox(options=options)
        elif browser_name == "edge":
            # Edge shares options with Chrome internally in many setups
            options = webdriver.EdgeOptions()
            if headless:
                options.add_argument("headless")
            driver = webdriver.Edge(options=options)
        else:
            raise ValueError(f"Browser '{browser_name}' is not supported by the factory!")
        # Global configurations applied to ALL browsers
        driver.maximize_window()
        driver.implicitly_wait(10)
        return driver
```

---

## 3. Integrating the Factory with PyTest

Now that our Factory handles all the dirty work of configuring options and capabilities, our PyTest `conftest.py` file becomes incredibly clean.

Better yet, we can use PyTest's `pytest_addoption` hook to let the user pass the browser choice directly from the command line!

**conftest.py**

```python
import pytest
from core.driver_factory import DriverFactory
# 1. Add custom command line arguments to PyTest
def pytest_addoption(parser):
    parser.addoption("--browser", action="store", default="chrome", help="Browser to run tests on")
    parser.addoption("--headless", action="store_true", default=False, help="Run in headless mode")
# 2. Capture the CLI arguments in a fixture
@pytest.fixture(scope="session")
def browser(request):
    return request.config.getoption("--browser")
@pytest.fixture(scope="session")
def headless(request):
    return request.config.getoption("--headless")
# 3. Use the Driver Factory!
@pytest.fixture
def driver(browser, headless):
    # We ask the Factory for a driver, we don't care how it makes it!
    driver_instance = DriverFactory.get_driver(browser, headless)
    yield driver_instance
    driver_instance.quit()
```

---

## 4. Execution Output

Now, our test suite is completely dynamic. The developer or the CI/CD pipeline can control the entire framework execution just by changing the terminal command!

Let's run a simple `test_login.py` script.

### Running standard Chrome:

```bash
pytest test_login.py
```
```text
============================= test session starts ==============================
test_login.py::test_successful_login PASSED                              [100%]
============================== 1 passed in 4.22s ===============================
```

### Running Headless Firefox from the terminal:

```bash
pytest test_login.py --browser=firefox --headless
```
```text
============================= test session starts ==============================
test_login.py::test_successful_login PASSED                              [100%]
============================== 1 passed in 6.14s ===============================
```

## Conclusion

The Driver Factory pattern is essential for scaling. 
- The `conftest.py` file is responsible for **Test Lifecycle** (Setup/Teardown).
- The `driver_factory.py` file is responsible for **Browser Configuration**.

By separating these concerns, you can easily add support for Remote WebDrivers (Selenium Grid, BrowserStack) into the factory later without ever touching your PyTest code!

In our next article, we will look at how to generate complex, randomized test data using the **Builder Pattern**!
