---
title: Enterprise Framework Architecture in Selenium Python
date: 20-Jan-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [selenium, python, framework, architecture, folder-structure, enterprise]
category: Framework Architecture
categories: [Framework Architecture, Selenium, Python]
excerpt: >-
  Stop throwing all your Python files into one folder! Discover the industry-standard folder structure for an Enterprise Selenium framework, perfectly separating Pages, Tests, Utils, and Data.
readTime: 6 min read
---

# Enterprise Framework Architecture in Selenium Python

Over the last 10 articles, we have introduced a multitude of advanced concepts: The Page Object Model, Driver Factories, Data-Driven Testing, Configuration Managers, and Logging modules. 

If you just throw all of these `.py` files into a single directory, your project will become an unreadable mess. 

To build a true **Enterprise Automation Suite**, everything must be organized into a highly strict, modular folder structure. In this article, we will outline the industry-standard architecture for a Selenium Python PyTest framework!

---

## 1. The Standard Folder Structure

Here is the blueprint for a professional automation framework. Notice how every component has a dedicated folder based on its specific responsibility.

```text
mycodeyatra-framework/
│
├── core/                   # 1. Core Engine
│   ├── driver_factory.py
│   └── custom_logger.py
│
├── utils/                  # 2. Reusable Utilities
│   ├── webdriver_utils.py  # (Base Page)
│   ├── config_reader.py
│   └── data_utils.py
│
├── pages/                  # 3. Page Object Model
│   ├── base_page.py
│   ├── login_page.py
│   └── dashboard_page.py
│
├── testdata/               # 4. External Data
│   ├── users.csv
│   └── environments.json
│
├── tests/                  # 5. Actual PyTest Files
│   ├── conftest.py
│   ├── test_login.py
│   └── test_checkout.py
│
├── logs/                   # 6. Generated Assets (GitIgnored)
│   └── automation_2026.log
│
├── reports/                # 7. Test Results (GitIgnored)
│   └── report.html
│
├── config.ini              # 8. Environment Configurations
├── requirements.txt        # 9. Python Dependencies
└── pytest.ini              # 10. PyTest Run Configurations
```

Let's break down why this structure is so important.

---

## 2. Breaking Down the Components

### 1. The `core/` Directory
The `core` directory contains the low-level engine of your framework. 
Code in this folder rarely changes. It is responsible for instantiating WebDrivers via the `DriverFactory` and generating the `CustomLogger`. No test logic or UI locators should ever exist here.

### 2. The `utils/` Directory
The `utils` directory contains reusable helper functions. 
The most important file here is `webdriver_utils.py` (which wraps raw Selenium clicks/waits). We also store `data_utils.py` (for generating random emails/passwords) and `config_reader.py` (for parsing `.ini` files). 

### 3. The `pages/` Directory (POM)
This is the heart of the Page Object Model. 
Every physical page in your application gets its own `.py` file here. These files store `Locators` (By.ID) and `Actions` (click_login). **No PyTest Assertions are allowed in this folder!**

### 4. The `testdata/` Directory
If you are doing Data-Driven Testing (DDT), you must separate your data from your code. This folder holds your `users.csv`, `products.xlsx`, and `api_payloads.json` files. 

### 5. The `tests/` Directory
This is where PyTest actually lives. 
Files here must start with `test_*.py`. Tests in this folder should be incredibly small—they simply initialize a Page Object, pass in Data, and make an `assert`. 
The `conftest.py` file also lives here to handle PyTest fixtures and setup/teardown hooks.

### 6. The Root Configuration Files
- `config.ini`: Holds URL and DB credentials for QA, Staging, and Prod.
- `requirements.txt`: Lists your pip dependencies (`selenium`, `pytest`, `faker`) so other developers can instantly install them via `pip install -r requirements.txt`.
- `pytest.ini`: Configures default PyTest behavior (e.g., automatically generating HTML reports or running in parallel).

---

## 3. Why Architecture Matters

If your framework is structured correctly, the flow of data is strictly regulated.

**The Golden Rule of Dependencies:**
`Tests` call `Pages`. 
`Pages` call `Utils`. 
`Utils` call `Core`. 

**A `Page` should NEVER call a `Test`. A `Core` file should NEVER call a `Page`.** 

When you enforce these architectural boundaries, your framework becomes infinitely scalable. You can have 5 engineers writing 1,000 tests without ever causing merge conflicts or circular import errors.

---

## 4. Execution Output

When your framework is organized this way, running the entire suite is as simple as pointing PyTest to the `tests/` folder from the root directory!

```bash
cd mycodeyatra-framework
pytest tests/ --env=QA --browser=chrome
```

```text
============================= test session starts ==============================
platform win32 -- Python 3.12.0, pytest-8.0.0
rootdir: C:\Automation\mycodeyatra-framework
configfile: pytest.ini
collected 150 items
tests/test_login.py ........                                             [  5%]
tests/test_dashboard.py ....................                             [ 18%]
tests/test_checkout.py ................................................. [ 51%]
...
============================= 150 passed in 45.22s =============================
```

## Conclusion

A tool is only as good as the architecture supporting it. While Selenium provides the raw commands to interact with a browser, the Folder Structure is what transforms those commands into an Enterprise Software Project.

By strictly separating your Data, Pages, Tests, and Utilities, you guarantee that your framework will survive application redesigns, team scaling, and thousands of daily CI/CD executions!

In our next article, we will dive deeper into the `conftest.py` file and explore advanced PyTest Hooks!
