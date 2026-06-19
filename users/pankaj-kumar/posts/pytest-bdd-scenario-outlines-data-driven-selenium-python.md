---
title: Data-Driven BDD: Scenario Outlines in pytest-bdd
date: 15-Mar-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [python, selenium, bdd, pytest-bdd, scenario-outlines, gherkin, data-driven-testing]
category: Selenium Python
categories: [Selenium Python, Behavior Driven Development]
excerpt: >-
  Why write four scenarios when you only need one? Learn how to use Scenario Outlines and Examples tables to achieve massive data-driven test coverage in pytest-bdd.
readTime: 4 min read
---

# Data-Driven BDD: Scenario Outlines in pytest-bdd

In our previous tutorial, we learned how to use Data Tables to pass multiple rows of data into a *single step* (like adding 5 items to a shopping cart). 

But what if you want to run an *entire scenario* multiple times with different data? 

For example, imagine you want to test a login form. You want to test a valid user, an invalid username, an invalid password, and a locked-out user. Writing four separate scenarios would result in massive duplication of the exact same Gherkin steps.

This is where **Scenario Outlines** shine! They allow you to execute the exact same Scenario structure dozens of times by injecting rows from an `Examples:` table.

---

## 1. Writing a Scenario Outline in Gherkin

A Scenario Outline acts as a template. Instead of hardcoding values like `"admin"`, you use angle brackets `<variable_name>` to indicate that the value should be injected dynamically.

At the bottom of the scenario, you provide an `Examples:` table containing the data. 

```gherkin
Feature: User Login Combinations
  Scenario Outline: Attempting login with various credentials
    Given I navigate to the login page
    When I enter the username "<username>" and password "<password>"
    Then I should see the message "<expected_msg>"
    Examples:
      | username      | password      | expected_msg              |
      | admin         | secret123     | Welcome, Admin!           |
      | invalid_user  | secret123     | User not found            |
      | admin         | bad_password  | Incorrect password        |
      | locked_user   | secret123     | Account has been locked   |
```

When `pytest` reads this feature file, it will treat this single block of text as **four completely separate tests**, spinning up a new browser session for each row in the Examples table!

---

## 2. Parsing Scenario Outlines in Python

The beauty of `pytest-bdd` is that Scenario Outlines do not require any complex parsing utilities (unlike Data Tables). 

Because the data is injected into the angle brackets `<username>` *before* the step definition is called, `pytest-bdd` simply reads the injected value as a standard string parameter!

Here is how you write the Step Definitions:

```python
from pytest_bdd import scenario, given, when, then, parsers
# Notice we bind to 'Scenario Outline' exactly as written in the feature file
@scenario('login.feature', 'Attempting login with various credentials')
def test_login_combinations():
    pass
@given("I navigate to the login page")
def navigate_to_login(driver):
    driver.get("https://practice.mycodeyatra.com/login")
# The parsers.parse string must match the Gherkin text EXACTLY, 
# including the double quotes if you used them!
@when(parsers.parse('I enter the username "{username}" and password "{password}"'))
def enter_credentials(driver, username, password):
    driver.find_element("id", "username").send_keys(username)
    driver.find_element("id", "password").send_keys(password)
    driver.find_element("id", "submit").click()
@then(parsers.parse('I should see the message "{expected_msg}"'))
def verify_message(driver, expected_msg):
    actual_message = driver.find_element("id", "alert-msg").text
    assert expected_msg in actual_message
```

### How it works:
1. Pytest reads the first row of the Examples table: `admin`, `secret123`, `Welcome, Admin!`.
2. It replaces `<username>` with `admin` in the Gherkin step.
3. The step becomes `When I enter the username "admin" and password "secret123"`.
4. Our Python `parsers.parse` intercepts this string, extracts `admin` and `secret123`, and passes them to our `enter_credentials` function!
5. Once the scenario finishes, Pytest loops back and starts again with the second row!

---

## 3. Best Practices for Scenario Outlines

### Keep Examples Focused
Do not put 50 rows in a single Examples table. If your login test has 50 permutations, it will take hours to run because a Selenium browser boots up for *every single row*. Keep the rows limited to the most critical business edge cases.

### Use Descriptive Columns
The columns in your `Examples:` table should be self-documenting. If a new QA engineer reads the feature file, they should instantly understand why a specific row exists.

```gherkin
    Examples:
      | test_case          | username | password  | expected_msg       |
      | Valid Admin        | admin    | secret123 | Welcome, Admin!    |
      | SQL Injection      | ' OR 1=1 | secret123 | Invalid characters |
      | Trailing Whitespace| admin    | secret123 | Welcome, Admin!    |
```
*Note: In the above example, we added a `test_case` column just for readability! We don't even have to bind it to a step definition; it simply makes the Gherkin file easier for a human to read.*

## Conclusion

Scenario Outlines are the ultimate tool for Data-Driven Testing in BDD. By combining them with `pytest` parameterization, you can achieve massive test coverage with a very small amount of code.

In the next tutorial, we will explore **BDD Reporting**—learning how to generate beautiful, human-readable HTML reports that showcase every Gherkin step execution for your management team!
