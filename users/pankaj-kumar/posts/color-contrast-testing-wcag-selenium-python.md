---
title: Color Contrast Testing: Automating WCAG 1.4.3 Guidelines in Selenium Python
date: 21-Feb-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [python, selenium, accessibility, color-contrast, wcag, ui-testing]
category: Selenium Python
categories: [Selenium Python, Accessibility]
excerpt: >-
  Stop squinting at your screen. Learn the math behind Relative Luminance and how to extract CSS rgba values to computationally assert WCAG color contrast ratios natively in Selenium Python.
readTime: 4 min read
---

# Color Contrast Testing: Automating WCAG 1.4.3 Guidelines in Selenium Python

Have you ever visited a website with light gray text on a white background and squinted to read it? Poor color contrast is one of the most common accessibility failures on the internet, directly alienating users with visual impairments or color blindness.

The Web Content Accessibility Guidelines (WCAG) specifically mandate that normal text must have a minimum contrast ratio of **4.5:1**, and large text must have a ratio of **3:1**.

In this tutorial, we will learn how to extract CSS colors using Selenium Python and computationally evaluate their contrast ratio to ensure compliance with WCAG 1.4.3!

---

## 1. Extracting CSS Colors using Selenium

When you request a CSS color property via Selenium's `value_of_css_property()`, it typically returns the color in an `rgba(R, G, B, A)` string format. 

Let's look at how to retrieve the foreground (text) and background colors of an element:

```python
from selenium import webdriver
from selenium.webdriver.common.by import By
driver = webdriver.Chrome()
driver.get("https://practice.mycodeyatra.com/buttons")
# Locate a primary CTA button
cta_btn = driver.find_element(By.ID, "primary-cta")
# Extract the CSS colors
text_color_rgba = cta_btn.value_of_css_property("color")
bg_color_rgba = cta_btn.value_of_css_property("background-color")
print(f"Text: {text_color_rgba}")   # e.g., 'rgba(255, 255, 255, 1)'
print(f"Background: {bg_color_rgba}") # e.g., 'rgba(0, 123, 255, 1)'
```

---

## 2. Converting RGBA Strings to RGB Tuples

To mathematically calculate the contrast ratio, we need to extract the raw Red, Green, and Blue integers from that string. 

We can write a simple regex or string manipulation utility:

```python
import re
def extract_rgb(rgba_string: str) -> tuple:
    """
    Converts 'rgba(255, 255, 255, 1)' to (255, 255, 255)
    """
    # Find all digits in the string
    numbers = re.findall(r'\d+', rgba_string)
    # Return the first three as integers (R, G, B)
    return int(numbers[0]), int(numbers[1]), int(numbers[2])
# Usage
text_rgb = extract_rgb("rgba(255, 255, 255, 1)") # (255, 255, 255)
bg_rgb = extract_rgb("rgba(0, 123, 255, 1)")     # (0, 123, 255)
```

---

## 3. The Math: Calculating Relative Luminance

According to WCAG, the contrast ratio is calculated using **Relative Luminance**. The formula is slightly complex because human eyes do not perceive all colors equally (we are much more sensitive to green than to blue).

Here is the exact Python implementation of the WCAG luminance formula:

```python
def calculate_luminance(r: int, g: int, b: int) -> float:
    """
    Calculates the relative luminance of a color according to WCAG 2.0.
    """
    # 1. Normalize the RGB values to fractions of 1
    a = [r / 255.0, g / 255.0, b / 255.0]
    # 2. Apply the WCAG gamma correction formula
    for i in range(3):
        if a[i] <= 0.03928:
            a[i] = a[i] / 12.92
        else:
            a[i] = ((a[i] + 0.055) / 1.055) ** 2.4
    # 3. Calculate final luminance with human perception weights
    return 0.2126 * a[0] + 0.7152 * a[1] + 0.0722 * a[2]
```

---

## 4. Calculating the Final Contrast Ratio

Once we have the relative luminance of both the foreground text (`L1`) and the background (`L2`), we can calculate the final ratio. 

*(Note: The WCAG formula requires that `L1` is always the lighter color, meaning it has the higher luminance value).*

```python
def get_contrast_ratio(rgb1: tuple, rgb2: tuple) -> float:
    """
    Calculates the contrast ratio between two RGB tuples.
    Returns a float between 1.0 and 21.0
    """
    lum1 = calculate_luminance(*rgb1)
    lum2 = calculate_luminance(*rgb2)
    # L1 must be the lighter color (higher luminance)
    L1 = max(lum1, lum2)
    L2 = min(lum1, lum2)
    # WCAG Contrast Ratio Formula
    ratio = (L1 + 0.05) / (L2 + 0.05)
    return round(ratio, 2)
```

---

## 5. Integrating it into a Pytest Assertion

Now we have a fully functional utility to mathematically prove whether a button is legally readable! Let's wrap it in a test:

```python
import pytest
from selenium import webdriver
from selenium.webdriver.common.by import By
class TestColorContrast:
    def test_primary_button_contrast(self):
        driver = webdriver.Chrome()
        driver.get("https://practice.mycodeyatra.com/buttons")
        btn = driver.find_element(By.ID, "primary-cta")
        # Extract CSS
        fg_css = btn.value_of_css_property("color")
        bg_css = btn.value_of_css_property("background-color")
        # Parse to tuples
        fg_rgb = extract_rgb(fg_css)
        bg_rgb = extract_rgb(bg_css)
        # Calculate Ratio
        ratio = get_contrast_ratio(fg_rgb, bg_rgb)
        print(f"Contrast Ratio is {ratio}:1")
        # Assert WCAG 1.4.3 Compliance (Minimum 4.5:1 for normal text)
        assert ratio >= 4.5, f"WCAG Violation! Contrast ratio {ratio}:1 is too low!"
        driver.quit()
```

## Conclusion

Automating color contrast tests natively in Selenium without relying on external plugins is incredibly powerful. It allows you to run assertions on highly dynamic components (like asserting the color contrast of a button *while* it is being actively hovered).

In the next tutorial, we will zoom out and look at how to generate comprehensive **Accessibility Reports** to share with your Product and Design teams!
