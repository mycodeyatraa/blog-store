---
title: ARIA Validation: Automating Screen Reader Compatibility in Selenium Python
date: 18-Feb-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [python, selenium, accessibility, aria, screen-readers, ui-testing, wcag]
category: Selenium Python
categories: [Selenium Python, Accessibility]
excerpt: >-
  When your UI changes state, screen readers need to know. Learn how to automate the assertion of aria-expanded, aria-hidden, and aria-invalid attributes using Selenium Python.
readTime: 4 min read
---

# ARIA Validation: Automating Screen Reader Compatibility in Selenium Python

While tools like Axe-Core are excellent at catching static accessibility issues (like missing `alt` attributes or poor contrast), they cannot effectively validate dynamic state changes.

When a user interacts with a complex UI component—such as a custom dropdown, a modal dialog, or an expanding accordion—how does a screen reader know what is happening? The answer is **ARIA (Accessible Rich Internet Applications)** attributes.

In this tutorial, we will learn how to automate ARIA state validation using Selenium Python to ensure your complex UI components remain accessible.

---

## 1. What are ARIA Attributes?

HTML5 provides native semantic elements like `<button>` and `<input>` which screen readers understand automatically. However, modern web apps often build custom components using generic `<div>` and `<span>` elements. 

To bridge this gap, developers use ARIA attributes. There are three main types:
- **Roles:** Define what an element is (`role="dialog"`, `role="tab"`).
- **Properties:** Give elements extra meaning (`aria-label="Close"`, `aria-required="true"`).
- **States:** Define dynamic conditions (`aria-expanded="true"`, `aria-hidden="false"`).

As Quality Engineers, we must automate the assertion of these **States** during UI interactions.

---

## 2. Automating `aria-expanded` (Accordions & Dropdowns)

When an accordion or custom dropdown is clicked, its state toggles. Visually, a panel appears. Accessibly, the `aria-expanded` attribute must flip from `false` to `true`.

Let's automate this validation:

```python
import pytest
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
class TestAriaValidation:
    @pytest.fixture(autouse=True)
    def setup(self):
        self.driver = webdriver.Chrome()
        self.driver.implicitly_wait(10)
        yield
        self.driver.quit()
    def test_accordion_aria_expanded(self):
        self.driver.get("https://practice.mycodeyatra.com/accordion")
        # Locate the accordion trigger button
        accordion_btn = self.driver.find_element(By.ID, "faq-trigger-1")
        # 1. Assert initial closed state
        assert accordion_btn.get_attribute("aria-expanded") == "false", \
            "Accordion should initially be collapsed for screen readers"
        # 2. Interact with the component
        accordion_btn.click()
        # 3. Assert dynamic state change
        # Wait for any CSS transitions to finish
        WebDriverWait(self.driver, 5).until(
            lambda d: accordion_btn.get_attribute("aria-expanded") == "true"
        )
        # 4. Verify the panel itself is no longer hidden
        panel = self.driver.find_element(By.ID, "faq-panel-1")
        assert panel.get_attribute("aria-hidden") == "false" \
            or panel.get_attribute("aria-hidden") is None
```

---

## 3. Automating `aria-invalid` (Form Validation)

When a user submits a form with missing or incorrect data, visual text like "Invalid Email" appears. But a screen reader relies on the `aria-invalid` attribute being applied to the input field itself.

```python
    def test_form_validation_aria_invalid(self):
        self.driver.get("https://practice.mycodeyatra.com/register")
        email_input = self.driver.find_element(By.ID, "email")
        submit_btn = self.driver.find_element(By.ID, "submit")
        # Initial state: Not invalid
        assert email_input.get_attribute("aria-invalid") in [None, "false"]
        # Enter bad data and submit
        email_input.send_keys("invalid-email")
        submit_btn.click()
        # Assert the ARIA state instantly flags the field as invalid
        WebDriverWait(self.driver, 5).until(
            lambda d: email_input.get_attribute("aria-invalid") == "true"
        )
        # Verify the error message is linked via aria-describedby
        error_msg_id = email_input.get_attribute("aria-describedby")
        assert error_msg_id is not None, "Screen reader needs to know which error message to read!"
        error_msg_element = self.driver.find_element(By.ID, error_msg_id)
        assert "Please enter a valid email" in error_msg_element.text
```

---

## 4. Automating `aria-hidden` (Modals)

When a modal dialog opens, the background page should be hidden from screen readers using `aria-hidden="true"`. Otherwise, visually impaired users will get trapped navigating the background page instead of the modal.

```python
    def test_modal_trapping_aria_hidden(self):
        self.driver.get("https://practice.mycodeyatra.com/modal")
        main_content = self.driver.find_element(By.ID, "main-wrapper")
        modal_dialog = self.driver.find_element(By.ID, "promo-modal")
        open_modal_btn = self.driver.find_element(By.ID, "open-modal")
        # Modal opens
        open_modal_btn.click()
        # Assert modal is visible to screen readers (role="dialog" and aria-hidden="false")
        assert modal_dialog.get_attribute("role") == "dialog"
        assert modal_dialog.get_attribute("aria-hidden") in [None, "false"]
        # CRITICAL: Assert the main background content is hidden from screen readers
        assert main_content.get_attribute("aria-hidden") == "true", \
            "Background content was not hidden! Screen reader users will get lost."
```

## Conclusion

Automated ARIA validation pushes your framework beyond basic functional testing. By explicitly asserting `aria-expanded`, `aria-invalid`, `aria-hidden`, and `aria-describedby` during state transitions, you guarantee that your application's complex interactivity remains 100% accessible to assistive technologies.

In our next blog, we will explore **Color Contrast Testing**—learning how to computationally evaluate HEX and RGB values using Python!
