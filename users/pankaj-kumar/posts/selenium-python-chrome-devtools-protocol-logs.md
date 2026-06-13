---
title: Chrome DevTools Protocol (CDP) and Logs in Selenium Python
date: 24-Nov-2024
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [selenium, python, cdp, logs, network-throttling, devtools]
category: UI Automation
categories: [UI Automation, Selenium, Python]
excerpt: >-
  Unlock God-mode in your browser. Learn how to use Selenium 4's Chrome DevTools Protocol (CDP) in Python to capture silent JavaScript console errors, throttle network speeds, and monitor API traffic.
readTime: 5 min read
---

# Chrome DevTools Protocol (CDP) and Logs in Selenium Python

In the early days of test automation, Selenium was strictly a "black-box" testing tool. It could only do what a human could do: click buttons, type text, and read elements on the screen.

If a Javascript error occurred in the console, Selenium couldn't see it. If an API request failed in the background, Selenium couldn't detect it unless the UI explicitly showed an error.

With the release of Selenium 4, everything changed. Selenium introduced native support for the **Chrome DevTools Protocol (CDP)**, granting Python engineers God-mode access to the browser's internal engine.

---

## 1. W3C Protocol vs CDP

As we discussed in Blog 4, Selenium traditionally communicates via the **W3C WebDriver Protocol** (a standard HTTP REST API). This is a stateless connection.

**CDP (Chrome DevTools Protocol)**, on the other hand, operates over a persistent **WebSocket**. This allows for **Bidirectional (BiDi)** communication. Instead of Python constantly asking the browser "Did anything happen?", the browser can actively push events (like a Network Error or a Console Log) directly to your Python script in real-time!

---

## 2. Capturing Browser Console Logs

Have you ever had a test pass, but the application was actually fundamentally broken because a React component crashed silently in the background?

With CDP, we can extract the exact logs you see when you press F12 in Chrome.

```python
from selenium import webdriver
def test_console_logs():
    driver = webdriver.Chrome()
    driver.get("https://mycodeyatra.com")
    # Execute some script that causes an error intentionally
    driver.execute_script("console.error('Simulated React crash!');")
    # Retrieve all logs of type 'browser'
    logs = driver.get_log("browser")
    for entry in logs:
        print(f"[{entry['level']}] {entry['message']}")
        # We can even assert that no SEVERE errors occurred!
        assert entry['level'] != 'SEVERE', f"Test failed due to JS Error: {entry['message']}"
    driver.quit()
```

---

## 3. Emulating Network Conditions (Throttling)

Does your loading spinner actually work? It is impossible to test loading spinners if your tests are running on a gigabit connection because the page loads in 0.1 seconds.

Using CDP, we can tell the browser engine to artificially throttle its own network speed, simulating a terrible 3G mobile connection.

```python
from selenium import webdriver
def test_slow_network():
    driver = webdriver.Chrome()
    # We must cast the driver to access CDP commands natively
    # Define our throttling parameters
    network_conditions = {
        "offline": False,
        "latency": 100,      # Add 100ms of latency
        "download_throughput": 50 * 1024,  # 50 kbps
        "upload_throughput": 50 * 1024     # 50 kbps
    }
    # Execute the CDP Command
    driver.execute_cdp_cmd("Network.emulateNetworkConditions", network_conditions)
    # Now, this navigation will be incredibly slow!
    driver.get("https://mycodeyatra.com")
    # Here you could assert that your loading spinner is displayed
    driver.quit()
```

---

## 4. Capturing Network Performance Metrics

Want to know exactly how long a specific API call took during your UI test? By enabling the Performance logging preference, we can capture raw CDP network events.

```python
from selenium import webdriver
from selenium.webdriver.chrome.options import Options
def test_network_performance():
    # 1. We must explicitly enable performance logging in ChromeOptions
    options = Options()
    options.set_capability("goog:loggingPrefs", {"performance": "ALL"})
    driver = webdriver.Chrome(options=options)
    driver.get("https://mycodeyatra.com")
    # 2. Retrieve performance logs
    perf_logs = driver.get_log("performance")
    # 3. Parse the JSON strings
    import json
    for log in perf_logs:
        message = json.loads(log["message"])["message"]
        # Look specifically for Network responses
        if message["method"] == "Network.responseReceived":
            url = message["params"]["response"]["url"]
            status = message["params"]["response"]["status"]
            print(f"Loaded {url} with status {status}")
    driver.quit()
```

## Conclusion

By unlocking the power of the Chrome DevTools Protocol, your Python Selenium framework can look beyond the DOM. You can catch silent JavaScript exceptions, simulate real-world mobile network conditions, and monitor underlying API traffic.

However, capturing errors isn't the only way to debug. Sometimes, a picture is worth a thousand words. In our final article of Phase 1, we will learn how to capture **Screenshots and Videos** upon test failure!
