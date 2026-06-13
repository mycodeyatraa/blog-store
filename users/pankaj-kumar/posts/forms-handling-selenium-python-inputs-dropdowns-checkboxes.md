---
title: Forms Handling in Selenium Python: Inputs, Dropdowns, and Checkboxes
date: 03-Dec-2024
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [selenium, python, forms, dropdowns, checkboxes, select-class]
category: UI Automation
categories: [UI Automation, Selenium, Python]
excerpt: >-
  Master HTML forms with Selenium Python. Learn how to clear text inputs, safely toggle checkboxes, and use the powerful Select class to automate native dropdowns effortlessly.
readTime: 5 min read
---

# Forms Handling in Selenium Python: Inputs, Dropdowns, and Checkboxes

Forms are the backbone of any web application. Whether a user is logging in, registering, or submitting a payment, they are interacting with an HTML form.

In our previous End-to-End test, we briefly saw how to type into an input field and click a submit button. However, modern forms contain much more than just text inputs.

In this article, we will learn how to automate **Text Inputs, Checkboxes, Radio Buttons, and Native Dropdowns** using Selenium Python!

---

## 1. Handling Text Inputs

We already know that we use the `send_keys()` method to type text. But what if the input field already has text in it (like a pre-filled email address)?

If you call `send_keys()` on a pre-filled field, Selenium will simply *append* the new text to the end of the old text!

To handle this, we must always `clear()` the field before typing.

```python
from selenium import webdriver
from selenium.webdriver.common.by import By
def test_text_inputs():
    driver = webdriver.Chrome()
    driver.get("https://mycodeyatra.com/profile")
    email_input = driver.find_element(By.ID, "user-email")
    # 1. Clear any existing text
    email_input.clear()
    # 2. Type the new text
    email_input.send_keys("new_email@mycodeyatra.com")
    # 3. (Optional) Read the text back to verify
    # Remember: For inputs, you cannot use .text. You must use .get_attribute("value")
    actual_value = email_input.get_attribute("value")
    assert actual_value == "new_email@mycodeyatra.com"
```

---

## 2. Handling Checkboxes and Radio Buttons

Checkboxes and Radio buttons are just `<input>` elements with a specific type (`type="checkbox"` or `type="radio"`). 

Because they are inputs, we interact with them using the `click()` method!

However, clicking blindly is dangerous. If a checkbox is *already* checked, calling `.click()` will uncheck it! We must always verify the state first using `is_selected()`.

```python
def test_checkboxes_and_radios():
    driver = webdriver.Chrome()
    driver.get("https://mycodeyatra.com/settings")
    # --- CHECKBOX ---
    newsletter_checkbox = driver.find_element(By.ID, "subscribe-newsletter")
    # Only click it if it is NOT already selected
    if not newsletter_checkbox.is_selected():
        newsletter_checkbox.click()
    assert newsletter_checkbox.is_selected() is True, "Checkbox failed to check!"
    # --- RADIO BUTTON ---
    # Radio buttons usually share the same 'name' attribute
    theme_radios = driver.find_elements(By.NAME, "theme-preference")
    for radio in theme_radios:
        if radio.get_attribute("value") == "dark-mode":
            if not radio.is_selected():
                radio.click()
            break
```

---

## 3. Handling Native Dropdowns (`<select>`)

Native HTML dropdowns use the `<select>` tag containing multiple `<option>` tags.

You *could* automate this by clicking the `<select>` to open it, and then clicking the `<option>`. But Selenium provides a much cleaner, specialized utility class called `Select`.

```python
from selenium.webdriver.support.ui import Select
def test_native_dropdown():
    driver = webdriver.Chrome()
    driver.get("https://mycodeyatra.com/register")
    # 1. Locate the actual <select> element
    country_dropdown_element = driver.find_element(By.ID, "country-select")
    # 2. Pass the element to the Select class
    country_select = Select(country_dropdown_element)
    # 3. Now you have three powerful ways to select an option!
    # Method A: Select by the visible text (Most common & readable)
    country_select.select_by_visible_text("India")
    # Method B: Select by the underlying 'value' attribute
    # <option value="US">United States</option>
    country_select.select_by_value("US")
    # Method C: Select by Index (0-based)
    # Selects the 3rd option in the list
    country_select.select_by_index(2)
    # 4. Verify the selection
    selected_option = country_select.first_selected_option
    assert selected_option.text == "Canada"
```

### What about Custom Dropdowns?
If the dropdown is built using `<div>` and `<ul>` tags instead of a `<select>` tag (which is very common in modern React/Angular apps), the `Select` class **will not work**. 

For custom dropdowns, you must fall back to the basic strategy:
1. `driver.find_element(By.ID, "custom-dropdown-container").click()`
2. `driver.find_element(By.XPATH, "//li[text()='India']").click()`

## Conclusion

Interacting with forms requires careful state management. 
- Always `clear()` text inputs before typing.
- Always check `is_selected()` before clicking a checkbox.
- And always use the `Select` class for native dropdowns to keep your code clean!

Now that you can fill out any form, the next step is dealing with popups! In our next article, we will learn how to handle **Alerts, Confirms, and Prompts**.
