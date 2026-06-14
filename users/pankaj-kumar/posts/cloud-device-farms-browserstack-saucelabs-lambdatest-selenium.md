---
title: Cloud Device Farms: BrowserStack, Sauce Labs, and LambdaTest
date: 28-Jun-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [python, selenium, browserstack, saucelabs, lambdatest, cloud-testing]
category: Cloud Grids & Advanced DevOps
categories: [Cloud Grids & Advanced DevOps, Python, Automation]
excerpt: >-
  You can't put an iPhone in a Docker container! Learn how to run your Selenium Python tests on thousands of physical devices using enterprise cloud providers like BrowserStack, Sauce Labs, and LambdaTest.
readTime: 6 min read
---

# Cloud Device Farms: BrowserStack, Sauce Labs, and LambdaTest

In our previous article, we learned how to build our own Kubernetes cluster to scale Chrome containers. But eventually, you will face a harsh reality: **You cannot put an iPhone in a Docker container.** 

When your Product Manager says, *"We need to ensure the checkout flow works on Safari on an actual iPhone 15 Pro, and on Microsoft Edge on Windows 11,"* your Kubernetes cluster becomes useless.

To achieve true cross-browser and cross-device compatibility, Enterprise teams turn to **Cloud Device Farms**. In this article, we will teach you how to point your Python automation framework to the "Big Three" cloud providers: BrowserStack, Sauce Labs, and LambdaTest!

---

## 1. What is a Cloud Device Farm?

A Cloud Device Farm is a massive warehouse filled with thousands of real smartphones, tablets, and desktop computers. 

Instead of maintaining these devices yourself, you rent them by the minute. Your Pytest framework sends standard Selenium commands to a secure Cloud URL, and the provider routes your commands to an actual, physical device sitting in their warehouse!

---

## 2. Integrating with BrowserStack

BrowserStack is the industry leader for cross-browser testing. To connect to BrowserStack, you need your **Username** and **Access Key**.

Instead of `webdriver.ChromeOptions()`, we use a specific `bstack:options` dictionary to tell BrowserStack exactly which device and OS we want to rent.

**tests/conftest_browserstack.py**

```python
import os
import pytest
from selenium import webdriver
from selenium.webdriver.chrome.options import Options as ChromeOptions
@pytest.fixture(scope="function")
def browserstack_driver():
    # 1. Fetch credentials securely
    BS_USER = os.getenv("BROWSERSTACK_USERNAME")
    BS_KEY = os.getenv("BROWSERSTACK_ACCESS_KEY")
    # 2. Define the Cloud Configuration Matrix
    bstack_options = {
        "osVersion" : "17",
        "deviceName" : "iPhone 15 Pro Max",
        "realMobile" : "true",
        "projectName" : "MyCodeYatra E-Commerce",
        "buildName" : "Nightly Release v2.1",
        "sessionName" : "Checkout Flow Validation",
    }
    options = ChromeOptions() # Or SafariOptions if targeting Safari
    options.set_capability('bstack:options', bstack_options)
    # 3. Connect to the BrowserStack Hub
    remote_url = f"https://{BS_USER}:{BS_KEY}@hub-cloud.browserstack.com/wd/hub"
    print(f"\n[Cloud] Requesting iPhone 15 Pro Max from BrowserStack...")
    driver = webdriver.Remote(command_executor=remote_url, options=options)
    yield driver
    # 4. Mark test as passed/failed in the BrowserStack UI
    driver.execute_script('browserstack_executor: {"action": "setSessionStatus", "arguments": {"status":"passed", "reason": "Test Successful!"}}')
    driver.quit()
```

---

## 3. Integrating with Sauce Labs

Sauce Labs is a massive enterprise competitor to BrowserStack, famous for its security and enterprise integrations. The syntax is almost identical, but Sauce Labs uses a `sauce:options` capability dictionary.

**tests/conftest_saucelabs.py**

```python
import os
import pytest
from selenium import webdriver
from selenium.webdriver.edge.options import Options as EdgeOptions
@pytest.fixture(scope="function")
def saucelabs_driver():
    SAUCE_USER = os.getenv("SAUCE_USERNAME")
    SAUCE_KEY = os.getenv("SAUCE_ACCESS_KEY")
    # Requesting Windows 11 with Microsoft Edge!
    sauce_options = {
        "build": "Nightly Release v2.1",
        "name": "Windows 11 Edge Validation",
        "screenResolution": "1920x1080"
    }
    options = EdgeOptions()
    options.browser_version = "latest"
    options.platform_name = "Windows 11"
    options.set_capability('sauce:options', sauce_options)
    # Sauce Labs Data Center URL (US-West in this example)
    remote_url = f"https://{SAUCE_USER}:{SAUCE_KEY}@ondemand.us-west-1.saucelabs.com:443/wd/hub"
    driver = webdriver.Remote(command_executor=remote_url, options=options)
    yield driver
    # Update Sauce Labs Status
    driver.execute_script("sauce:job-result=passed")
    driver.quit()
```

---

## 4. Integrating with LambdaTest

LambdaTest is a newer, highly performant alternative that is rapidly gaining popularity due to its speed and competitive pricing. They use a `LT:Options` dictionary.

**tests/conftest_lambdatest.py**

```python
import os
import pytest
from selenium import webdriver
from selenium.webdriver.chrome.options import Options as ChromeOptions
@pytest.fixture(scope="function")
def lambdatest_driver():
    LT_USER = os.getenv("LT_USERNAME")
    LT_KEY = os.getenv("LT_ACCESS_KEY")
    lt_options = {
        "build": "Nightly Release v2.1",
        "name": "macOS Safari Validation",
        "platformName": "macOS Sonoma",
        "browserName": "Safari",
        "browserVersion": "17.0",
        "visual": True, # Capture Step-by-Step Screenshots
        "video": True,  # Record a Video of the Execution!
        "network": True # Capture Network Logs
    }
    options = ChromeOptions()
    options.set_capability('LT:Options', lt_options)
    remote_url = f"https://{LT_USER}:{LT_KEY}@hub.lambdatest.com/wd/hub"
    driver = webdriver.Remote(command_executor=remote_url, options=options)
    yield driver
    driver.execute_script("lambda-status=passed")
    driver.quit()
```

### The Cloud Configuration Matrix
To achieve true Enterprise maturity, you shouldn't hardcode these dictionaries. You should create a `config_matrix.json` file containing hundreds of device combinations. 

Your Pytest suite should iterate through this Matrix, dynamically passing the device configurations into your fixture using `@pytest.mark.parametrize`!

## Conclusion

By moving your execution to Cloud Device Farms, you unlock infinite cross-browser compatibility without buying a single physical device!
- **BrowserStack:** Use `bstack:options` to access the largest real-device lab in the world.
- **Sauce Labs:** Use `sauce:options` to execute on highly secure enterprise VMs.
- **LambdaTest:** Use `LT:Options` for blazing fast, cost-effective cloud execution with built-in video recording.
- Use `webdriver.Remote()` to connect to the cloud hubs, and don't forget to inject JavaScript (`execute_script`) to mark the test as Passed/Failed on the provider's dashboard!

In Phase 15, we will enter the final frontier of Test Automation: **AI, LangChain, and Autonomous Test Agents!**
