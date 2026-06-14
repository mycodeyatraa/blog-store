---
title: Responsive Visual Testing: Asserting Mobile and Tablet Viewports in Python
date: 12-Apr-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [python, selenium, visual-testing, responsive, mobile, pytest]
category: Visual Testing
categories: [Visual Testing, Python, UI Automation]
excerpt: >-
  Stop testing only on desktop monitors! Learn how to use Pytest parametrize to dynamically resize your Selenium browser and capture visual baselines for Mobile, Tablet, and Desktop viewports in a single script.
readTime: 6 min read
---

# Responsive Visual Testing: Asserting Mobile and Tablet Viewports in Python

Over 60% of modern web traffic comes from mobile devices. If your QA strategy only tests the application on a massive 1920x1080 Desktop monitor, you are completely blind to the majority of your users' experiences.

When developers use CSS Media Queries (e.g., `@media (max-width: 768px)`), they completely change the architecture of the site. Menus collapse into "Hamburgers," images stack vertically, and buttons resize.

In this article, we will teach you how to write a single Pytest script that automatically iterates through Desktop, Tablet, and Mobile viewports, capturing precise Visual Baselines for every screen size!

---

## 1. The Viewport Matrix

We do not want to write three separate tests for Desktop, Tablet, and Mobile. That violates the DRY (Don't Repeat Yourself) principle.

Instead, we will use `@pytest.mark.parametrize` to feed a list of viewport dimensions into a single test function!

Here are the industry standard viewport resolutions:
- **Desktop (1080p):** 1920x1080
- **Tablet (iPad Portrait):** 768x1024
- **Mobile (iPhone 13):** 390x844

---

## 2. Writing the Responsive Visual Test

Let's write a script that iterates through our viewport matrix, explicitly setting `driver.set_window_size()`, and dynamically naming the screenshot based on the device!

**tests/test_responsive_visuals.py**

```python
import os
import pytest
from selenium import webdriver
from PIL import Image
from pixelmatch.contrib.PIL import pixelmatch
# Define our Viewport Matrix
VIEWPORTS = [
    ("Desktop", 1920, 1080),
    ("Tablet", 768, 1024),
    ("Mobile", 390, 844)
]
@pytest.fixture(scope="function")
def driver():
    # Setup Chrome
    options = webdriver.ChromeOptions()
    options.add_argument("--headless=new") # Run headless for faster execution
    driver_instance = webdriver.Chrome(options=options)
    yield driver_instance
    driver_instance.quit()
@pytest.mark.parametrize("device_name, width, height", VIEWPORTS)
def test_responsive_homepage(driver, device_name, width, height):
    # 1. Resize the browser to simulate the specific device
    driver.set_window_size(width, height)
    # 2. Navigate to the application
    driver.get("https://httpbin.org/")
    # 3. Capture the current state, naming it dynamically!
    current_image = f"current_homepage_{device_name}.png"
    print(f"\n[Visual] Capturing {device_name} viewport ({width}x{height})...")
    driver.save_screenshot(current_image)
    # 4. Perform Visual Comparison (Assuming baselines are already stored locally)
    baseline_image = f"baseline_homepage_{device_name}.png"
    # If the baseline doesn't exist, we save the current image as the new golden baseline!
    if not os.path.exists(baseline_image):
        print(f"⚠️ No baseline found for {device_name}! Establishing new golden baseline.")
        os.rename(current_image, baseline_image)
        assert True
        return
    # 5. Compare the Pixels
    print(f"[Visual] Comparing {device_name} against the Golden Baseline...")
    img_baseline = Image.open(baseline_image)
    img_current = Image.open(current_image)
    img_diff = Image.new("RGBA", img_baseline.size)
    mismatch = pixelmatch(img_baseline, img_current, img_diff, threshold=0.1)
    # Cleanup
    os.remove(current_image)
    assert mismatch < 50, f"UI Regression on {device_name}! {mismatch} pixels changed."
    print(f"✅ {device_name} Viewport is pixel-perfect!")
```

---

## 3. Execution Output

Let's execute our new responsive suite. Notice how `pytest` automatically runs the test three times, resizing the browser window mathematically for each execution!

```bash
pytest test_responsive_visuals.py -s
```

```text
============================= test session starts ==============================
platform win32 -- Python 3.12.0, pytest-8.0.0
rootdir: C:\Automation\mycodeyatra-visual
collected 3 items
test_responsive_visuals.py 
[Visual] Capturing Desktop viewport (1920x1080)...
⚠️ No baseline found for Desktop! Establishing new golden baseline.
.
[Visual] Capturing Tablet viewport (768x1024)...
⚠️ No baseline found for Tablet! Establishing new golden baseline.
.
[Visual] Capturing Mobile viewport (390x844)...
⚠️ No baseline found for Mobile! Establishing new golden baseline.
.
============================== 3 passed in 8.42s ===============================
```

If we immediately run the suite a second time, it will compare against the newly established baselines:

```text
============================= test session starts ==============================
collected 3 items
test_responsive_visuals.py 
[Visual] Capturing Desktop viewport (1920x1080)...
[Visual] Comparing Desktop against the Golden Baseline...
✅ Desktop Viewport is pixel-perfect!
.
[Visual] Capturing Tablet viewport (768x1024)...
[Visual] Comparing Tablet against the Golden Baseline...
✅ Tablet Viewport is pixel-perfect!
.
[Visual] Capturing Mobile viewport (390x844)...
[Visual] Comparing Mobile against the Golden Baseline...
✅ Mobile Viewport is pixel-perfect!
.
============================== 3 passed in 9.11s ===============================
```

## Conclusion

A successful modern automation framework must test responsive design!
- Use `@pytest.mark.parametrize` to cleanly feed viewport tuples into a single test.
- Use `driver.set_window_size(width, height)` to trigger the application's CSS Media Queries.
- Dynamically name your baselines based on the device name (e.g., `baseline_Mobile.png`) to ensure you are always comparing apples to apples!

However, `set_window_size` only changes the *size* of the screen. What if the website uses JavaScript to check if the user is *actually* on an iPhone by reading the `User-Agent`? 

In our next article, we will teach you how to use **Chrome DevTools Emulation** to trick the browser into genuinely believing it is running on a physical Mobile Device!
