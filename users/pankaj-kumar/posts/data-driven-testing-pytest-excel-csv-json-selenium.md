---
title: Data-Driven Testing in Pytest: Excel, CSV, and JSON
date: 10-Jul-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [python, selenium, pytest, data-driven, excel, json]
category: Pytest Framework
categories: [Pytest Framework, Python, Automation]
excerpt: >-
  Stop duplicating your test functions! Learn how to use @pytest.mark.parametrize to execute a single test scenario hundreds of times by dynamically reading test data from Excel, CSV, and JSON files.
readTime: 6 min read
---

# Data-Driven Testing in Pytest: Excel, CSV, and JSON

Imagine you need to test a Login page. You don't just want to test one valid user. You want to test:
1. Valid user, valid password
2. Valid user, invalid password
3. Invalid user, valid password
4. Blank username, blank password
5. SQL Injection payload

If you write a separate Pytest function for each of these 5 scenarios, you are duplicating code and violating the **DRY (Don't Repeat Yourself)** principle. 

Instead, you should write **ONE** Pytest function, and feed it a list of 5 different test data sets. This is called **Data-Driven Testing (DDT)**. In this article, we will teach you how to decouple your test data from your test logic using Pytest's `@pytest.mark.parametrize` and external files!

---

## 1. The `@pytest.mark.parametrize` Decorator

The easiest way to make a test data-driven is to use Pytest's built-in `parametrize` decorator. You provide a comma-separated string of variable names, and a list of tuples containing the data.

**tests/test_login_ddt.py**

```python
import pytest
from selenium import webdriver
from selenium.webdriver.common.by import By
# Our list of Test Data
login_data = [
    ("admin", "Pass123", "Dashboard"),
    ("admin", "wrong", "Invalid credentials"),
    ("baduser", "Pass123", "Invalid credentials"),
    ("", "", "Username required")
]
@pytest.mark.parametrize("username, password, expected_message", login_data)
def test_login_scenarios(username, password, expected_message):
    driver = webdriver.Chrome()
    driver.get("https://mycodeyatra.com/login")
    driver.find_element(By.ID, "user").send_keys(username)
    driver.find_element(By.ID, "pass").send_keys(password)
    driver.find_element(By.ID, "login-btn").click()
    body_text = driver.find_element(By.TAG_NAME, "body").text
    assert expected_message in body_text
    driver.quit()
```

If you run this, Pytest will automatically execute `test_login_scenarios` **four times**, passing a different set of credentials each time! 

---

## 2. Reading Test Data from Excel (openpyxl)

Hardcoding `login_data` inside the Python file is still bad practice. Business Analysts and Manual QA engineers prefer to manage test data in Excel files.

Let's read our test data dynamically from an Excel file using the `openpyxl` library.

**Installation:**

```bash
pip install openpyxl
```

**utils/excel_reader.py**

```python
import openpyxl
def get_data_from_excel(file_path, sheet_name):
    workbook = openpyxl.load_workbook(file_path)
    sheet = workbook[sheet_name]
    data = []
    # Skip the header row (start from row 2)
    for row in range(2, sheet.max_row + 1):
        username = sheet.cell(row, 1).value
        password = sheet.cell(row, 2).value
        expected = sheet.cell(row, 3).value
        # Avoid appending empty rows
        if username is not None:
            data.append((username, password, expected))
    return data
```

**tests/test_login_excel.py**

```python
import pytest
from utils.excel_reader import get_data_from_excel
# Fetch data dynamically before the test runs!
excel_data = get_data_from_excel("data/LoginData.xlsx", "Sheet1")
@pytest.mark.parametrize("username, password, expected_message", excel_data)
def test_login_excel(username, password, expected_message):
    print(f"\nTesting: {username} / {password} -> Expecting: {expected_message}")
    # ... Selenium logic here ...
```

---

## 3. Reading Test Data from CSV

CSV (Comma Separated Values) files are lighter and faster to read than Excel files. We can use Python's built-in `csv` module.

**data/login.csv**

```csv
username,password,expected
admin,Pass123,Dashboard
invalid,wrong,Error
```

**utils/csv_reader.py**

```python
import csv
def get_data_from_csv(file_path):
    data = []
    with open(file_path, mode='r') as file:
        reader = csv.reader(file)
        next(reader) # Skip the header row
        for row in reader:
            data.append(tuple(row)) # Convert list to tuple
    return data
```

---

## 4. Reading Test Data from JSON

For complex data structures (like API payloads or nested user profiles), JSON is the industry standard. Python has a built-in `json` module.

**data/users.json**

```json
[
  {"username": "admin", "password": "123", "expected": "Dashboard"},
  {"username": "bad", "password": "bad", "expected": "Error"}
]
```

**utils/json_reader.py**

```python
import json
def get_data_from_json(file_path):
    with open(file_path, mode='r') as file:
        json_data = json.load(file)
    data = []
    for item in json_data:
        data.append((item["username"], item["password"], item["expected"]))
    return data
```

## Conclusion

Data-Driven Testing (DDT) is mandatory for any Enterprise Framework.
- It prevents code duplication and keeps your test functions clean.
- Use `@pytest.mark.parametrize` to loop over your data.
- Never hardcode data in your test file! Extract it to **Excel** (using `openpyxl`), **CSV**, or **JSON** files so non-technical stakeholders can easily add new test scenarios without touching your Python code.

In our next article, we will go back and cover **Authenticated Browser Sessions**, teaching you how to inject Cookies and LocalStorage to bypass login screens entirely!
