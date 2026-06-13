---
title: Building a Logging Framework in Selenium Python
date: 18-Jan-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [selenium, python, logging, framework, architecture, pytest]
category: Framework Architecture
categories: [Framework Architecture, Selenium, Python]
excerpt: >-
  Stop using print() statements! Learn how to integrate Python's built-in logging module to generate timestamped, persistent execution logs for your enterprise automation suite.
readTime: 6 min read
---

# Building a Logging Framework in Selenium Python

In the previous articles, we built an elegant Utility Framework using standard Python `print()` statements.

```python
print(f"Successfully clicked element: {locator}")
```

While `print()` is great for local debugging, it is completely useless in an enterprise CI/CD pipeline. Why?
1. It does not provide a timestamp.
2. It does not provide severity levels (INFO vs ERROR vs DEBUG).
3. It does not automatically write to an external file. If your Jenkins pipeline crashes, your console output might be lost forever!

To solve this, we must replace all `print()` statements with a professional **Logging Framework** using Python's built-in `logging` module.

---

## 1. Creating the Custom Logger

Python's `logging` module is incredibly powerful but requires a bit of setup. We want our logger to simultaneously print to the console *and* write to an `automation.log` file.

Let's create a new utility class!

**utils/custom_logger.py**

```python
import logging
import os
from datetime import datetime
class CustomLogger:
    @staticmethod
    def get_logger(name="AutomationFramework"):
        """
        Creates a custom logger that outputs to both Console and a File.
        """
        logger = logging.getLogger(name)
        # If the logger already has handlers, return it to prevent duplicate logs
        if logger.hasHandlers():
            return logger
        logger.setLevel(logging.DEBUG)
        # 1. Create a dynamic log file name with today's date
        log_dir = os.path.join(os.path.dirname(os.path.dirname(os.path.abspath(__file__))), 'logs')
        os.makedirs(log_dir, exist_ok=True)
        date_str = datetime.now().strftime("%Y-%m-%d")
        log_file = os.path.join(log_dir, f"automation_{date_str}.log")
        # 2. Create Handlers (File and Console)
        file_handler = logging.FileHandler(log_file)
        console_handler = logging.StreamHandler()
        # 3. Create a beautiful Log Format: [DATE] [LEVEL] [LOGGER] Message
        formatter = logging.Formatter('%(asctime)s - [%(levelname)s] - [%(name)s] : %(message)s')
        file_handler.setFormatter(formatter)
        console_handler.setFormatter(formatter)
        # 4. Attach the Handlers to the Logger
        logger.addHandler(file_handler)
        logger.addHandler(console_handler)
        return logger
```

---

## 2. Integrating the Logger into the Base Page

Now that we have a professional logging engine, let's inject it into our `WebDriverUtils` (Base Page) class. 

We will replace every single `print()` statement with `self.log.info()`, and any exceptions with `self.log.error()`!

**utils/webdriver_utils.py**

```python
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from selenium.common.exceptions import TimeoutException
from utils.custom_logger import CustomLogger
class WebDriverUtils:
    def __init__(self, driver, default_timeout=10):
        self.driver = driver
        self.timeout = default_timeout
        # Initialize the Logger!
        self.log = CustomLogger.get_logger("WebDriverUtils")
    def click_element(self, locator):
        try:
            wait = WebDriverWait(self.driver, self.timeout)
            element = wait.until(EC.element_to_be_clickable(locator))
            element.click()
            # USE THE LOGGER INSTEAD OF PRINT!
            self.log.info(f"Successfully clicked element: {locator}")
        except Exception as e:
            # LOG THE EXACT EXCEPTION AS AN ERROR!
            self.log.error(f"Failed to click element: {locator}. Error: {str(e)}")
            raise e
    def enter_text(self, locator, text):
        wait = WebDriverWait(self.driver, self.timeout)
        element = wait.until(EC.visibility_of_element_located(locator))
        element.clear()
        element.send_keys(text)
        # For security, we might not want to log passwords! 
        # But for this example, we will log the text.
        self.log.info(f"Entered text '{text}' into {locator}")
```

---

## 3. Using the Logger inside PyTest

You can also use the Logger directly inside your PyTest test files to log Business-level steps (e.g., "Starting the login process").

**tests/test_login.py**

```python
from pages.login_page import LoginPage
from utils.custom_logger import CustomLogger
# Create a logger specific to this test file!
log = CustomLogger.get_logger("TestLogin")
def test_secure_login(driver):
    log.info("--- Starting Test: test_secure_login ---")
    login_page = LoginPage(driver)
    log.info("Attempting to login with admin credentials")
    login_page.login("admin_user", "SecurePass123!")
    log.info("Verifying Dashboard title")
    assert "Dashboard" in driver.title
    log.info("--- Test Completed Successfully ---")
```

---

## 4. Execution Output

When we execute our test suite, the console output is completely transformed. 

Instead of simple strings, we get beautiful, enterprise-grade logs complete with exact millisecond timestamps and severity levels!

```text
============================= test session starts ==============================
test_login.py::test_secure_login 
2026-06-13 18:20:10,145 - [INFO] - [TestLogin] : --- Starting Test: test_secure_login ---
2026-06-13 18:20:10,145 - [INFO] - [TestLogin] : Attempting to login with admin credentials
2026-06-13 18:20:10,250 - [INFO] - [WebDriverUtils] : Entered text 'admin_user' into ('id', 'username')
2026-06-13 18:20:10,312 - [INFO] - [WebDriverUtils] : Entered text 'SecurePass123!' into ('id', 'password')
2026-06-13 18:20:10,500 - [INFO] - [WebDriverUtils] : Successfully clicked element: ('id', 'login-btn')
2026-06-13 18:20:11,100 - [INFO] - [TestLogin] : Verifying Dashboard title
2026-06-13 18:20:11,105 - [INFO] - [TestLogin] : --- Test Completed Successfully ---
PASSED
============================== 1 passed in 4.50s ===============================
```

Additionally, if you check your project folder, you will see a new `logs/automation_2026-06-13.log` file containing the exact same output. You can now easily attach this log file to Jira tickets or Allure Reports!

## Conclusion

By replacing `print()` with Python's `logging` module, you unlock:
1. **Traceability:** Exact timestamps for every UI action.
2. **Severity:** Differentiating between `INFO`, `DEBUG`, `WARNING`, and `ERROR`.
3. **Persistence:** Saving all execution history to local `.log` files for debugging CI/CD failures.

In our next article, we will combine the Page Object Model, Driver Factories, Config Managers, and Loggers together into an **Enterprise Framework Architecture Folder Structure!**
