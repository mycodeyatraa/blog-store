---
title: Data Driven Testing in Selenium Python using PyTest
date: 27-Dec-2024
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [selenium, python, data-driven-testing, pytest, parametrize, excel]
category: Framework Architecture
categories: [Framework Architecture, Selenium, Python]
excerpt: >-
  Stop duplicating your test code! Learn how to build Data-Driven frameworks in Selenium Python by reading test data from CSV, JSON, and Excel files using PyTest's parametrize decorator.
readTime: 6 min read
---

# Data-Driven Testing in Selenium Python using PyTest

Welcome to **Phase 3: Test Design & Framework Architecture**!

Up until now, our test scripts have been hardcoded. If we wanted to test a login page with 5 different user accounts (Admin, Customer, Invalid User, Blank Password, Locked Account), we would have to write 5 separate test functions. 

Writing redundant code is the enemy of automation. 

In this article, we will learn about **Data-Driven Testing (DDT)**. We will separate our test logic from our test data, allowing us to run a single test function dozens of times automatically using data from CSVs, JSONs, or Excel files.

---

## 1. The `@pytest.mark.parametrize` Decorator

PyTest provides a built-in decorator specifically designed for data-driven testing: `@pytest.mark.parametrize`. 

Instead of hardcoding the username and password inside the test, we pass them as arguments to the test function.

```python
import pytest
from selenium import webdriver
from selenium.webdriver.common.by import By
# 1. Provide the data directly in the decorator
@pytest.mark.parametrize("username, password, expected_msg", [
    ("admin_user", "Secure123!", "Welcome, Admin"),
    ("customer", "Pass123", "Welcome, Customer"),
    ("invalid", "wrong", "Invalid credentials"),
    ("", "blank", "Username is required")
])
def test_login(username, password, expected_msg):
    driver = webdriver.Chrome()
    driver.get("https://mycodeyatra.com/login")
    # 2. Use the parameterized variables
    driver.find_element(By.ID, "user").send_keys(username)
    driver.find_element(By.ID, "pass").send_keys(password)
    driver.find_element(By.ID, "login-btn").click()
    # 3. Assert using the parameterized expected result
    actual_msg = driver.find_element(By.ID, "toast-message").text
    assert actual_msg == expected_msg
    driver.quit()
```

When you run this script, PyTest automatically executes `test_login` **four times**, injecting a different row of data each time!

---

## 2. Reading Data from a CSV File

Hardcoding data in the Python file is better than duplicating tests, but it still mixes code with data. For true DDT, we should store our data externally.

Let's read data from a `users.csv` file using Python's built-in `csv` module.

**users.csv**

```csv
username,password,expected_msg
admin_user,Secure123!,Welcome, Admin
invalid,wrong,Invalid credentials
```

**test_login_csv.py**

```python
import csv
import pytest
from selenium import webdriver
# Helper function to read the CSV and return a list of tuples
def read_csv_data():
    data = []
    with open('users.csv', 'r') as file:
        reader = csv.reader(file)
        next(reader) # Skip the header row!
        for row in reader:
            data.append(tuple(row))
    return data
# Pass the helper function to the parametrize decorator!
@pytest.mark.parametrize("username, password, expected_msg", read_csv_data())
def test_login_from_csv(username, password, expected_msg):
    # ... exact same Selenium logic as before!
    print(f"Testing with: {username}")
```

---

## 3. Reading Data from a JSON File

JSON is the standard format for modern web APIs. If your test data is complex (e.g., nested lists or dictionaries), JSON is far superior to CSV.

**data.json**

```json
[
  {"user": "admin", "pass": "secret", "msg": "Success"},
  {"user": "guest", "pass": "123", "msg": "Denied"}
]
```

**test_login_json.py**

```python
import json
import pytest
def read_json_data():
    with open('data.json', 'r') as file:
        raw_data = json.load(file)
        # Convert list of dicts to list of tuples for PyTest
        return [(item['user'], item['pass'], item['msg']) for item in raw_data]
@pytest.mark.parametrize("user, pwd, msg", read_json_data())
def test_login_from_json(user, pwd, msg):
    print(f"Logging in {user} with {pwd} expecting {msg}")
    # ... Selenium logic
```

---

## 4. Reading Data from Excel

For enterprise environments where non-technical Business Analysts write test data, Excel (`.xlsx`) is the king. 

To read Excel files in Python, we need a third-party library called `openpyxl`.

```bash
pip install openpyxl
```

**test_login_excel.py**

```python
import openpyxl
import pytest
def read_excel_data():
    data = []
    workbook = openpyxl.load_workbook('TestData.xlsx')
    sheet = workbook.active
    # Iterate through rows starting from row 2 (skipping header)
    for i in range(2, sheet.max_row + 1):
        username = sheet.cell(row=i, column=1).value
        password = sheet.cell(row=i, column=2).value
        msg = sheet.cell(row=i, column=3).value
        data.append((username, password, msg))
    return data
@pytest.mark.parametrize("username, password, expected_msg", read_excel_data())
def test_login_from_excel(username, password, expected_msg):
    # ... Selenium logic
    pass
```

---

## 5. Execution Output

When we run these data-driven tests, PyTest dynamically generates unique test names for every single row of data!

```text
============================= test session starts ==============================
platform win32 -- Python 3.12.0, pytest-8.0.0
rootdir: C:\Automation\mycodeyatra-tests
collected 8 items
test_ddt.py::test_login[admin_user-Secure123!-Welcome, Admin] PASSED     [ 12%]
test_ddt.py::test_login[customer-Pass123-Welcome, Customer] PASSED       [ 25%]
test_ddt.py::test_login[invalid-wrong-Invalid credentials] PASSED        [ 37%]
test_ddt.py::test_login[-blank-Username is required] PASSED              [ 50%]
test_ddt.py::test_login_from_csv[admin_user-Secure123!-Welcome, Admin] PASSED [ 62%]
test_ddt.py::test_login_from_csv[invalid-wrong-Invalid credentials] PASSED [ 75%]
test_ddt.py::test_login_from_json[admin-secret-Success] PASSED           [ 87%]
test_ddt.py::test_login_from_json[guest-123-Denied] PASSED               [100%]
============================== 8 passed in 20.44s ==============================
```

## Conclusion

Data-Driven Testing drastically reduces code duplication. By separating your logic from your data, you can add 50 new test scenarios simply by adding rows to an Excel sheet—without writing a single line of Python code!

However, while our *data* is now separated, our *Selenium Locators* (`By.ID, "user"`) are still hardcoded inside the test function. 

In our next article, we will solve this final maintenance problem by introducing the most famous design pattern in UI automation: **The Page Object Model (POM)!**
