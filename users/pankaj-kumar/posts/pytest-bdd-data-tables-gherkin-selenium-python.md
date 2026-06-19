---
title: Injecting Massive Datasets: Data Tables in pytest-bdd
date: 12-Mar-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [python, selenium, bdd, pytest-bdd, data-tables, gherkin, data-driven-testing]
category: Selenium Python
categories: [Selenium Python, Behavior Driven Development]
excerpt: >-
  When one line of Gherkin isn't enough, inject multi-dimensional arrays directly into your steps. Learn how to parse Data Tables into Python dictionaries for massive data-driven tests.
readTime: 4 min read
---

# Injecting Massive Datasets: Data Tables in pytest-bdd

In our earlier tutorials, we learned how to use Step Parameters to pass simple strings (like a username or password) from a Gherkin `.feature` file directly into a Python Step Definition.

But what if you need to pass a massive amount of data? Imagine testing a shopping cart feature where you need to add five different items, each with a different quantity and price. Writing five separate `And I add...` steps would make your feature file incredibly tedious.

To solve this, Gherkin provides **Data Tables**—a way to inject multi-dimensional arrays directly into your steps!

---

## 1. Writing Data Tables in Gherkin

A Data Table is defined in a `.feature` file using pipe `|` characters to separate columns. 

Unlike a `Scenario Outline` (which runs the *entire scenario* multiple times), a Data Table is passed into a *single step* as an argument.

Let's look at an example feature file:

```gherkin
Feature: Shopping Cart Management
  Scenario: Adding multiple items to the cart
    Given I navigate to the store
    When I add the following items to my cart:
      | Item Name       | Quantity | Price |
      | Laptop Pro      | 1        | 1200  |
      | Wireless Mouse  | 2        | 45    |
      | Mechanical Keyboard | 1    | 150   |
    Then my cart total should be exactly 1440
```

This is incredibly readable for a Product Manager! But how do we parse this table in Python?

---

## 2. Parsing Data Tables in pytest-bdd

By default, `pytest-bdd` captures anything written underneath a step that uses the `|` syntax and passes it to the Step Definition as a raw string. 

Wait... a raw string? Yes! If we simply bind the step, the argument we receive is a single string containing all the pipes and newlines. 

We have to parse it ourselves. Thankfully, we can write a simple utility function to convert this raw Gherkin table string into a list of Python dictionaries!

### The Parser Utility

```python
def parse_gherkin_table(raw_table_string: str) -> list[dict]:
    """
    Converts a raw Gherkin table string into a list of dictionaries.
    """
    # Split the string by newlines and remove empty lines
    lines = [line.strip() for line in raw_table_string.strip().split('\n') if line.strip()]
    # Extract the headers from the first line
    headers = [col.strip() for col in lines[0].split('|') if col.strip()]
    data = []
    # Loop through the remaining data rows
    for row in lines[1:]:
        values = [col.strip() for col in row.split('|') if col.strip()]
        # Zip the headers and values together into a dictionary
        row_dict = dict(zip(headers, values))
        data.append(row_dict)
    return data
```

---

## 3. Implementing the Step Definition

Now that we have our parsing utility, we can bind the `@when` step and process the Data Table.

*Note: In `pytest-bdd`, to capture a multi-line string or Data Table that follows a step, we do NOT use `parsers.parse()`. The table is automatically passed as an argument with the exact same name as the step text!*

Wait, actually, modern `pytest-bdd` injects the table into an argument if you declare it in the step definition signature, OR you can use the `datatable` fixture. Let's look at the standard approach using `parsers.parse()` with a named argument for the step, which then implicitly captures the table if the step name doesn't have parameters.

Actually, the official `pytest-bdd` way to handle this is that the multiline text/table is passed to the step definition if you include an argument named exactly after the step, but formatted as a valid python identifier. However, the easier, modern way is using the `target_fixture` or just letting `pytest-bdd` pass it as a multiline string to a step parameter.

Let's use the explicit `parsers.parse` method and the `datatable` convention:

```python
from pytest_bdd import when
import pytest
@when("I add the following items to my cart:")
def add_items_to_cart(driver, step):
    """
    The 'step' argument is automatically injected by pytest-bdd.
    It contains metadata about the step, including the multi-line table!
    """
    # 1. Extract the raw table string from the step metadata
    raw_table = step.lines
    # In pytest-bdd 5.0+, the raw table is often found in step.multiline or step.data_table
    # Depending on your version, you might need to join step.lines
    if isinstance(raw_table, list):
        raw_table = "\n".join(raw_table)
    # 2. Parse the table using our utility
    items = parse_gherkin_table(raw_table)
    # 3. Iterate through the dictionaries and automate the UI!
    for item in items:
        name = item["Item Name"]
        qty = int(item["Quantity"])
        price = float(item["Price"])
        print(f"Adding {qty}x {name} at ${price} each...")
        # Selenium Automation
        search_box = driver.find_element("id", "search")
        search_box.clear()
        search_box.send_keys(name)
        # (Assume logic to select the item and add 'qty' to cart...)
```

*Pro Tip:* Because Gherkin Data Tables are parsed as strings, your Python code must explicitly cast the dictionary values to `int` or `float` if you plan to do math with them!

---

## 4. Scenario Outlines vs. Data Tables

It is very common to confuse `Scenario Outlines` (which use `Examples:` tables) with `Data Tables`. 

Here is the Golden Rule:
- **Use a Scenario Outline** when you want to run the *entire scenario from start to finish* multiple times with different data (e.g., testing 5 different invalid passwords).
- **Use a Data Table** when you want to pass a *list of data into a single step* during a single scenario execution (e.g., adding 5 different items to a cart in one session).

## Conclusion

Data Tables are an incredible tool for making your Gherkin feature files dense with data while keeping them highly readable. By writing a simple string parsing utility in Python, you can instantly convert Product Requirements into iterable dictionaries for your Selenium WebDriver to process!

In the next tutorial, we will cover exactly how **Scenario Outlines** work, and how they differ fundamentally from the Data Tables we learned today!
