---
title: Locators in Selenium Python: Finding Elements on the Page
date: 15-Nov-2024
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [selenium, python, locators, xpath, css-selectors, dom]
category: UI Automation
categories: [UI Automation, Selenium, Python]
excerpt: >-
  Master the DOM. Learn the hierarchy of Selenium locator strategies in Python, from blazing-fast IDs and CSS Selectors to powerful (but dangerous) relative XPaths.
readTime: 5 min read
---

# Locators in Selenium Python: Finding Elements on the Page

In order to click a button, type into a text box, or scrape text from a paragraph, Selenium first needs to know *where* that element is.

A web browser renders pages using the **DOM (Document Object Model)**—a massive tree of HTML tags. To traverse this tree and find specific elements, we use **Locators**.

Choosing the right locator strategy is the most important skill in UI automation. A good locator makes your test robust and immune to minor UI changes. A bad locator makes your test "flaky," causing it to randomly fail tomorrow just because a developer added a new `<div>` tag.

---

## 1. The `By` Class in Python

In Python, Selenium provides a dedicated `By` class to define locator strategies. Before you can use it, you must import it:

```python
from selenium.webdriver.common.by import By
```

You use this class inside the `find_element()` method like this:

```python
element = driver.find_element(By.ID, "login-button")
```

Let's break down the different strategies in order of preference!

---

## 2. The Gold Standard: ID and Name

If the frontend developers did their job correctly, interactive elements will have a unique `id` or `name` attribute. These are the fastest and most robust locators available.

```html
<!-- Example HTML -->
<input type="text" id="username_field" name="user_login">
```

**Python Code:**

```python
# Best: ID is guaranteed to be unique on the page
driver.find_element(By.ID, "username_field").send_keys("admin")
 
# Second Best: Name is usually unique for form inputs
driver.find_element(By.NAME, "user_login").send_keys("admin")
```

---

## 3. The Sweet Spot: CSS Selectors

If an element doesn't have an ID, your next best option is a **CSS Selector**. CSS Selectors are incredibly fast because they use the browser's native styling engine to locate elements.

You can use CSS to find elements by class, by attribute, or by combining multiple attributes.

```html
<!-- Example HTML -->
<button class="btn primary-btn" data-testid="submit-btn">Login</button>
```

**Python Code:**

```python
# Finding by Class Name (Notice the dot prefix in CSS)
driver.find_element(By.CSS_SELECTOR, ".primary-btn").click()
 
# Finding by a Custom Data Attribute (Highly recommended!)
driver.find_element(By.CSS_SELECTOR, "[data-testid='submit-btn']").click()
```

---

## 4. The Double-Edged Sword: XPath

**XPath (XML Path Language)** is the most powerful locator strategy. It allows you to traverse up and down the DOM tree, find elements based on their inner text, and locate elements relative to their siblings.

However, XPaths are slower to evaluate and are notoriously **brittle** if written poorly.

### Absolute vs Relative XPath
Never use **Absolute XPath** (starting from the root `html` tag). If a developer adds a single `<div>` anywhere in the hierarchy, your test will instantly break.

```python
# TERRIBLE (Absolute): Breaks immediately if UI layout changes
driver.find_element(By.XPATH, "/html/body/div[1]/div[2]/form/input")
 
# GREAT (Relative): Searches anywhere in the DOM for an input of type 'password'
driver.find_element(By.XPATH, "//input[@type='password']")
```

### Finding Elements by Text
XPath is the *only* strategy that allows you to locate an element directly by the text it contains on the screen.

```python
# Finds a button containing the exact text "Checkout"
driver.find_element(By.XPATH, "//button[text()='Checkout']").click()
```

---

## 5. Locating Links

If you specifically need to click on an `<a>` anchor tag, Selenium provides two dedicated locators: `LINK_TEXT` and `PARTIAL_LINK_TEXT`.

```html
<a href="https://mycodeyatra.com/forgot-password">Forgot your password?</a>
```

**Python Code:**

```python
# Exact Match
driver.find_element(By.LINK_TEXT, "Forgot your password?").click()
 
# Partial Match (Great for dynamic links)
driver.find_element(By.PARTIAL_LINK_TEXT, "Forgot").click()
```

---

## 6. Finding Multiple Elements

So far, we have used `find_element()` (singular). If multiple elements match the locator, it returns the *very first* one it finds.

But what if you want to count how many rows are in a table, or scrape the text from a list of products? Use `find_elements()` (plural). This returns a **Python List** of WebElements.

```python
# Find all table rows
rows = driver.find_elements(By.CSS_SELECTOR, "table tr")
 
print(f"Found {len(rows)} rows!")
 
# Loop through the list and print the text of each row
for row in rows:
    print(row.text)
```
*(Note: If `find_element` cannot find the target, it throws a `NoSuchElementException`. However, if `find_elements` cannot find anything, it simply returns an empty list `[]` without failing!)*

## Conclusion

A successful automation engineer knows exactly when to use which locator. 
- Always prefer `ID` or `data-testid`.
- Fall back to `CSS_SELECTOR` for complex queries.
- Use `XPATH` only when traversing up to a parent element or locating by text.

Now that we can locate elements, it is time to validate them! In our next article, we will introduce the **pytest** assertion framework, allowing us to definitively pass or fail our test cases.
