---
title: Configuration Management in Selenium Python Frameworks
date: 15-Jan-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [selenium, python, framework, configparser, environment, pytest]
category: Framework Architecture
categories: [Framework Architecture, Selenium, Python]
excerpt: >-
  Make your framework environment-agnostic! Learn how to use Python's configparser and PyTest CLI arguments to dynamically switch between QA, Staging, and Production environments.
readTime: 6 min read
---

# Configuration Management in Selenium Python Frameworks

In our framework so far, our PyTest tests navigate to a URL like this:

```python
def test_login(driver):
    driver.get("https://qa.mycodeyatra.com/login")
```

Hardcoding the URL works fine until your QA Manager asks you to run the entire test suite against the new `staging.mycodeyatra.com` environment. 
If your URLs are hardcoded across 500 test files, you are in big trouble.

To fix this, we need a **Configuration Manager**. We need the ability to pass an environment variable (like `--env=staging`) from the terminal, and have the framework automatically figure out which URLs and database credentials to use.

---

## 1. Using `.ini` Files for Configuration

Python has a built-in module called `configparser` which perfectly handles `.ini` (Initialization) files. 

Let's create a file named `config.ini` in the root of our framework. We will define multiple "Sections" for our different testing environments.

**config.ini**

```ini
[QA]
base_url = https://qa.mycodeyatra.com
db_host = qa-db.internal
db_user = qa_admin
timeout = 10
[STAGING]
base_url = https://staging.mycodeyatra.com
db_host = stage-db.internal
db_user = stage_admin
timeout = 15
[PROD]
base_url = https://mycodeyatra.com
db_host = prod-db.internal
db_user = readonly_admin
timeout = 20
```

---

## 2. Creating the ConfigReader Utility

Now, we need a Python utility class that can read this `.ini` file based on a provided environment name. 

**utils/config_reader.py**

```python
import configparser
import os
class ConfigReader:
    @staticmethod
    def get_config(env, key):
        """
        Reads a specific key from the config.ini file for a given environment.
        """
        # Get the absolute path to the config file
        base_dir = os.path.dirname(os.path.dirname(os.path.abspath(__file__)))
        config_path = os.path.join(base_dir, 'config.ini')
        # Initialize parser
        config = configparser.ConfigParser()
        config.read(config_path)
        # Ensure environment section exists!
        env = env.upper()
        if not config.has_section(env):
            raise ValueError(f"Environment '{env}' not found in config.ini")
        return config.get(env, key)
```

---

## 3. Wiring Configuration into PyTest

We want to tell PyTest *which* environment to run against via the command line. We use the `pytest_addoption` hook inside our `conftest.py` file to achieve this.

**conftest.py**

```python
import pytest
from core.driver_factory import DriverFactory
from utils.config_reader import ConfigReader
# 1. Add custom command line arguments to PyTest
def pytest_addoption(parser):
    parser.addoption("--env", action="store", default="QA", help="Environment: QA, STAGING, or PROD")
    parser.addoption("--browser", action="store", default="chrome", help="Browser to run tests on")
# 2. Capture the environment variable
@pytest.fixture(scope="session")
def env(request):
    return request.config.getoption("--env")
# 3. Create a fixture that provides the Base URL for the requested environment
@pytest.fixture
def base_url(env):
    return ConfigReader.get_config(env, "base_url")
# 4. Standard Driver Fixture
@pytest.fixture
def driver(request):
    browser = request.config.getoption("--browser")
    driver_instance = DriverFactory.get_driver(browser)
    yield driver_instance
    driver_instance.quit()
```

---

## 4. Writing Environment-Agnostic Tests

Now for the magic. We inject our new `base_url` fixture directly into our test!

**test_environment.py**

```python
def test_homepage_loads_correctly(driver, base_url):
    # Notice we do NOT hardcode the URL! We use the fixture variable!
    print(f"\nNavigating to environment URL: {base_url}")
    driver.get(base_url)
    assert "MyCodeYatra" in driver.title
```

### Execution Output

Look what happens when we run the exact same test script, but change the `--env` flag in the terminal!

#### Run against QA:

```bash
pytest test_environment.py --env=QA -s
```
```text
============================= test session starts ==============================
collected 1 item
test_environment.py 
Navigating to environment URL: https://qa.mycodeyatra.com
PASSED
============================== 1 passed in 3.42s ===============================
```

#### Run against STAGING:

```bash
pytest test_environment.py --env=STAGING -s
```
```text
============================= test session starts ==============================
collected 1 item
test_environment.py 
Navigating to environment URL: https://staging.mycodeyatra.com
PASSED
============================== 1 passed in 4.01s ===============================
```

## Conclusion

By implementing a **Configuration Manager**, your entire automation framework becomes "Environment Agnostic." 

You can instantly point your 500 test cases at QA, Staging, or even Production without changing a single line of Python code. This is an absolute necessity for integrating your framework into automated CI/CD pipelines.

In our next article, we will tackle the final piece of the core framework puzzle: **Building a Custom Python Logging Framework!**
