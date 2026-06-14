---
title: Mobile Web Emulation: Simulating iPhones via Chrome DevTools
date: 15-Apr-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [python, selenium, mobile, emulation, devtools, user-agent]
category: Visual Testing
categories: [Visual Testing, Python, UI Automation]
excerpt: >-
  Test the Mobile Web without Appium! Learn how to use Chrome DevTools mobileEmulation to spoof User-Agents, pixel ratios, and touch events directly inside your Pytest Selenium scripts.
readTime: 6 min read
---

# Mobile Web Emulation: Simulating iPhones via Chrome DevTools

In our previous article, we performed responsive testing by resizing the browser window (`driver.set_window_size()`). This forces the CSS Media Queries to trigger, but it does NOT actually make the browser behave like a mobile device.

If a modern web application relies on JavaScript to read the `User-Agent` (to detect if the user is on iOS or Android) or checks for touch-screen capabilities, simply resizing the window will fail.

To perform true Mobile Web testing without buying an expensive Appium Device Farm, we can use **Chrome DevTools Emulation**! In this article, we will configure Selenium to actively spoof an iPhone 13 Pro.

---

## 1. What is Mobile Emulation?

Chrome DevTools contains a powerful feature called "Device Mode". When activated, Chrome completely alters its internal architecture to mimic a mobile device:
1. **User-Agent Spoofing:** It sends network requests explicitly identifying itself as Safari on iOS.
2. **Device Pixel Ratio (DPR):** It emulates Retina displays.
3. **Touch Events:** It converts standard mouse clicks into mobile `touchstart` and `touchend` events!

We can activate this feature directly in Selenium using `ChromeOptions`.

---

## 2. Emulating an iPhone in Selenium

We will use the `mobileEmulation` experimental option. We can pass a specific `deviceName` dictionary, and Chrome will automatically apply the correct viewport, pixel ratio, and User-Agent!

**tests/test_mobile_emulation.py**

```python
from selenium import webdriver
from selenium.webdriver.chrome.options import Options
def test_iphone_emulation():
    # 1. Define the Mobile Emulation Dictionary
    # You can use standard names like 'iPhone 12 Pro', 'Pixel 5', or 'iPad Mini'
    mobile_emulation = {
        "deviceName": "iPhone 12 Pro"
    }
    # 2. Inject it into ChromeOptions
    chrome_options = Options()
    chrome_options.add_experimental_option("mobileEmulation", mobile_emulation)
    # 3. Launch the Emulated Browser
    print("\n[Emulation] Launching Chrome as an iPhone 12 Pro...")
    driver = webdriver.Chrome(options=chrome_options)
    # 4. Navigate to a site that checks User-Agents
    driver.get("https://httpbin.org/user-agent")
    # 5. Extract the User-Agent the server *actually* received
    detected_user_agent = driver.find_element("tag name", "pre").text
    print(f"[Server Response] {detected_user_agent}")
    # Assert that the server truly believes we are on an iPhone!
    assert "iPhone" in detected_user_agent
    assert "Mobile" in detected_user_agent
    print("✅ Successfully tricked the server into serving the Mobile Web architecture!")
    driver.quit()
```

---

## 3. Custom Device Emulation

What if you need to test a custom device that isn't built into Chrome's default list? You can define the exact metrics mathematically!

```python
def test_custom_device_emulation():
    # Define a completely custom device matrix
    custom_device = {
        "deviceMetrics": {
            "width": 360,
            "height": 640,
            "pixelRatio": 3.0,
            "touch": True # Enables touch events instead of clicks!
        },
        "userAgent": "Mozilla/5.0 (Linux; Android 14) AppleWebKit/537.36 MyCodeYatra-Custom-Browser"
    }
    options = Options()
    options.add_experimental_option("mobileEmulation", custom_device)
    driver = webdriver.Chrome(options=options)
    driver.get("https://httpbin.org/user-agent")
    body_text = driver.find_element("tag name", "body").text
    print(f"\n[Custom Emulation] User Agent: {body_text}")
    assert "MyCodeYatra-Custom-Browser" in body_text
    driver.quit()
```

---

## 4. Execution Output

Let's execute the suite. Watch as the server explicitly returns the iOS Safari User-Agent, even though we are running standard desktop Chrome on Windows!

```bash
pytest test_mobile_emulation.py -s
```

```text
============================= test session starts ==============================
platform win32 -- Python 3.12.0, pytest-8.0.0
rootdir: C:\Automation\mycodeyatra-visual
collected 2 items
test_mobile_emulation.py 
[Emulation] Launching Chrome as an iPhone 12 Pro...
[Server Response] {
  "user-agent": "Mozilla/5.0 (iPhone; CPU iPhone OS 13_2_3 like Mac OS X) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/13.0.3 Mobile/15E148 Safari/604.1"
}
✅ Successfully tricked the server into serving the Mobile Web architecture!
.
[Custom Emulation] User Agent: {
  "user-agent": "Mozilla/5.0 (Linux; Android 14) AppleWebKit/537.36 MyCodeYatra-Custom-Browser"
}
.
============================== 2 passed in 3.85s ===============================
```

## Conclusion

Resizing the window is for CSS verification. Mobile Emulation is for Application Logic verification.
- Always use `mobileEmulation = {"deviceName": "iPhone 12 Pro"}` when you need to trigger Mobile-specific JavaScript, such as touch events or mobile-only popups.
- By injecting this directly into `ChromeOptions`, you can perform 90% of your Mobile Web testing without ever having to configure the massive, complex Appium architecture!

In our next and final article for Phase 8, we will explore **Percy Visual Testing**. We will learn how to integrate BrowserStack's Percy SDK to capture DOM snapshots and perform Cloud Visual Diffs with zero infrastructure!
