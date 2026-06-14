---
title: Scenario Outlines and The Behave Framework: Data-Driven BDD
date: 22-May-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [python, selenium, bdd, behave, scenario-outline, data-driven]
category: Behavior Driven Development
categories: [Behavior Driven Development, Python, Automation]
excerpt: >-
  Unlock massive Data-Driven testing using Gherkin Scenario Outlines! Learn how to iterate over Examples tables, and compare pytest-bdd with the standalone Behave BDD framework.
readTime: 6 min read
---

# Scenario Outlines and The Behave Framework: Data-Driven BDD

In our previous articles, we learned how to map a basic Gherkin `Scenario` to Python Step Definitions using `pytest-bdd`. But what if you need to test a search bar with 15 different search queries? Writing 15 separate `Scenario` blocks in your feature file would be an unreadable nightmare!

In this final article of Phase 10, we will learn how to achieve massive Data-Driven BDD execution using **Scenario Outlines**. Finally, we will compare `pytest-bdd` to its biggest rival in the Python ecosystem: the **Behave** framework.

---

## 1. Scenario Outlines: Data-Driven Gherkin

A `Scenario Outline` allows you to define a single scenario and run it repeatedly using a table of dynamic data. 

To use it, you replace hardcoded values with variables inside `<brackets>`, and provide an `Examples` table at the bottom of the scenario.

**tests/features/search.feature**

```gherkin
Feature: Product Search Validation
  As a customer
  I want to search for products
  So that I can quickly find items to purchase
  Scenario Outline: Validating various search queries
    Given the user navigates to the store homepage
    When the user searches for the product "<query>"
    Then the search results page should display "<expected_text>"
    And the total results count should be greater than <min_results>
    Examples:
      | query       | expected_text        | min_results |
      | Laptop      | Results for Laptop   | 10          |
      | Smartphone  | Results for Phones   | 25          |
      | Desk Chair  | Ergonomic Chairs     | 5           |
```

In the background, the BDD runner will execute this exact same scenario **three distinct times**, injecting the values from the table into the Gherkin steps!

---

## 2. Parsing Scenario Outlines in pytest-bdd

Mapping a `Scenario Outline` to Python step definitions is remarkably similar to a standard scenario. We use the same `parsers.parse()` technique we learned earlier!

The framework automatically takes the column headers from the `Examples` table (`query`, `expected_text`, `min_results`) and passes them as string arguments into our Python functions.

**tests/step_defs/test_search_steps.py**

```python
from pytest_bdd import scenarios, given, when, then, parsers
from selenium.webdriver.common.by import By
# Load the Scenario Outline!
scenarios('../features/search.feature')
@given('the user navigates to the store homepage')
def navigate_home(driver):
    driver.get("https://mycodeyatra.com/store")
@when(parsers.parse('the user searches for the product "{query}"'))
def enter_search(driver, query):
    search_box = driver.find_element(By.ID, "search-bar")
    search_box.clear()
    search_box.send_keys(query)
    driver.find_element(By.ID, "search-submit").click()
@then(parsers.parse('the search results page should display "{expected_text}"'))
def verify_results_text(driver, expected_text):
    header = driver.find_element(By.ID, "search-header").text
    assert expected_text in header
@then(parsers.parse('the total results count should be greater than {min_results}'))
def verify_results_count(driver, min_results):
    # Notice we must cast min_results to an integer!
    count_text = driver.find_element(By.ID, "total-count").text
    assert int(count_text) > int(min_results)
```

By simply defining the variables in `<brackets>` inside your Gherkin string, `pytest-bdd` handles all the iteration logic automatically!

---

## 3. Introducing the Behave Framework

Throughout this module, we have used `pytest-bdd`. However, if you look at job descriptions for Python Automation Engineers, you will frequently see a framework called **Behave**.

### What is Behave?
Behave is a dedicated BDD framework for Python. Unlike `pytest-bdd`, which is built *on top* of Pytest as a plugin, Behave is a completely standalone execution engine.

### How is Behave different from pytest-bdd?

1. **Folder Structure Restrictions:** Behave forces you to use a very specific folder structure. Your feature files must be in a folder named `features/`, and your step definitions must be inside a subfolder named `features/steps/`.
2. **Context Object:** In `pytest-bdd`, we shared context between steps using a Pytest fixture. In Behave, a shared `context` object is automatically passed into *every single* step definition function by default.
3. **Execution Command:** You do not run Behave tests with the `pytest` command. You run them using the `behave` command.
4. **No Pytest Plugins:** Because Behave does not use Pytest, you **cannot** use Pytest plugins like `pytest-xdist` for parallel execution or `pytest-rerunfailures`! You have to find Behave-specific workarounds.

### Example Behave Step Definition:
Notice how the decorators and the `context` object differ slightly from `pytest-bdd`:

**features/steps/search_steps.py**

```python
from behave import given, when, then
from selenium.webdriver.common.by import By
@given('the user navigates to the store homepage')
def step_impl(context):
    context.driver.get("https://mycodeyatra.com/store")
@when('the user searches for the product "{query}"')
def step_impl(context, query):
    search_box = context.driver.find_element(By.ID, "search-bar")
    search_box.send_keys(query)
```

## Conclusion

Data-Driven BDD testing is the ultimate form of automation. It allows Product Managers to define extensive test permutations in plain English, while the Automation Engineer only writes the underlying code once!
- Use **Scenario Outline** and an **Examples** table to define permutations.
- Extract dynamic variables into your Python functions using `parsers.parse()`.
- Choose **`pytest-bdd`** if you want to leverage the massive ecosystem of Pytest plugins (parallel execution, robust reporting).
- Choose **Behave** if you want a strictly opinionated, isolated BDD architecture native to Python!

Congratulations! You have completed Phase 10! You now know how to bridge the communication gap using Behavior-Driven Development.
