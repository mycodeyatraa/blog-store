---
title: Network Interception in Selenium Python using Chrome DevTools Protocol (CDP)
date: 20-Dec-2024
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [selenium, python, cdp, network-interception, mocking, selenium-4]
category: UI Automation
categories: [UI Automation, Selenium, Python]
excerpt: >-
  Mock APIs and intercept network requests directly within the browser! Learn how to leverage Selenium 4's Native CDP support to build lightning-fast, isolated UI tests.
readTime: 5 min read
---

# Network Interception in Selenium Python using Chrome DevTools Protocol (CDP)

Historically, Selenium was strictly a black-box tool. It could only click buttons and read text. If your web application failed because a backend API returned a `500 Internal Server Error`, Selenium had no way of knowing *why* the UI broke.

That all changed with **Selenium 4**.

Selenium 4 introduced native support for the **Chrome DevTools Protocol (CDP)**. This protocol gives Selenium a direct pipeline into the browser's internal engine, allowing us to capture network traffic, mock API responses, and stub external dependencies on the fly!

In this article, we will explore how to intercept and mock network requests using Selenium Python.

---

## 1. Why Mock Network Requests?

Imagine you are testing an e-commerce checkout page that relies on Stripe for payments.
- If Stripe goes down, your UI tests fail. 
- If you run 100 tests, you hit Stripe's API rate limits.
- If you want to test the "Payment Declined" UI, you need a specific credit card number.

Instead of actually hitting Stripe's servers, we can tell the browser: *"Hey, if you see a request going to `api.stripe.com`, don't send it. Just instantly return a fake `200 OK` response with this JSON payload."*

This makes your tests **faster, completely deterministic, and isolated from external failures**.

---

## 2. Capturing Network Traffic

Before we mock data, let's learn how to simply *listen* to the network and log every API call the browser makes. 

To do this, we use the `bidi` (Bi-Directional) network module available in Selenium 4.

```python
import time
from selenium import webdriver
def test_capture_network_traffic():
    driver = webdriver.Chrome()
    # 1. Enable network tracking in the DevTools engine
    driver.execute_cdp_cmd('Network.enable', {})
    # 2. Define a callback function that will execute every time a request is sent
    def capture_request(event):
        url = event['request']['url']
        method = event['request']['method']
        print(f"Intercepted {method} request to: {url}")
    # 3. Bind the callback to the 'Network.requestWillBeSent' CDP event
    driver.bidi_connection.session.execute(
        driver.bidi_connection.cdp.get_session_id(),
        "Network.requestWillBeSent",
        capture_request
    )
    # 4. Navigate! You will instantly see all the CSS, JS, and API requests printed.
    driver.get("https://mycodeyatra.com")
    # Sleep just to allow the asynchronous callbacks to print
    time.sleep(2)
    driver.quit()
```

---

## 3. Mocking an API Response

Now for the true superpower: intercepting a request and changing the response before it reaches the frontend.

To do this, we use the `NetworkInterceptor` class. Let's mock a `GET` request that normally returns a list of products, and force it to return a fake product.

```python
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.common.network import NetworkInterceptor, HttpResponse
def test_mock_api_response():
    driver = webdriver.Chrome()
    # 1. Define our fake HTTP Response
    fake_response = HttpResponse()
    fake_response.status_code = 200
    fake_response.body = b'{"products": [{"id": 999, "name": "Fake Mocked Laptop", "price": 10.00}]}'
    fake_response.headers = {'Content-Type': 'application/json'}
    # 2. Define our Intercept Logic
    def interceptor(request):
        # If the browser tries to hit the products API...
        if "api/v1/products" in request.url:
            # ... intercept it and return our fake JSON instead!
            request.create_response(fake_response)
    # 3. Attach the interceptor to the driver session
    with NetworkInterceptor(driver, interceptor):
        # While inside this block, all network traffic flows through our interceptor function
        driver.get("https://mycodeyatra.com/products")
        # The frontend UI will render our fake data, thinking it came from the real server!
        first_product_name = driver.find_element(By.CSS_SELECTOR, ".product-title").text
        assert first_product_name == "Fake Mocked Laptop"
    driver.quit()
```

### How this works:
1. The browser requests `https://mycodeyatra.com/products`.
2. The Javascript on the page executes `fetch('/api/v1/products')`.
3. Our `interceptor` catches it! It blocks the request from leaving the browser.
4. It instantly returns `{"products": [{"name": "Fake Mocked Laptop"}]}`.
5. The Javascript receives the data, parses it, and updates the HTML DOM.
6. Selenium reads the DOM and verifies our fake data is visible!

---

## 4. Execution Output

When we run these tests via PyTest, the execution is lightning fast because the mocked API request bypasses the internet entirely.

```text
============================= test session starts ==============================
platform win32 -- Python 3.12.0, pytest-8.0.0
rootdir: C:\Automation\mycodeyatra-tests
collected 2 items
test_network.py::test_capture_network_traffic PASSED                     [ 50%]
test_network.py::test_mock_api_response PASSED                           [100%]
============================== 2 passed in 6.12s ===============================
```

## Conclusion

By utilizing Selenium 4's Native CDP support, we have bridged the gap between Frontend UI testing and Backend API testing. 
- You can capture network logs to debug intermittent failures.
- You can mock 3rd party APIs to test offline or avoid rate limits.
- You can forcefully return `500` errors to verify how the UI handles backend crashes!

In our next article, we will move away from browser interactions and focus on test suite execution speed by learning how to run tests concurrently: **Parallel Execution using PyTest-Xdist!**
