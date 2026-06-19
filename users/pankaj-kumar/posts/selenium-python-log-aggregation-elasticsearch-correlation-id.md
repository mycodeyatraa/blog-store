---
title: Stop Guessing: Log Aggregation in Selenium Python
date: 10-Apr-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [python, selenium, log-aggregation, elasticsearch, correlation-id, pytest, observability]
category: Selenium Python
categories: [Selenium Python, Monitoring & Observability]
excerpt: >-
  When a test fails, you shouldn't have to guess why. Learn how to inject Correlation IDs into Chrome and automatically fetch backend server logs from Elasticsearch directly into your Pytest HTML reports.
readTime: 5 min read
---

# Stop Guessing: Log Aggregation in Selenium Python

In the previous tutorial, we learned how to push execution metrics to Datadog so that the DevOps team can track our test execution speeds over time. 

But what happens when a test actually fails? The QA Engineer reads the Selenium stack trace: `TimeoutException: Element not found`. That tells the QA Engineer *what* happened in the browser, but it doesn't tell them *why* it happened on the server.

To figure out why, the QA Engineer has to message a Backend Developer, who then manually searches through AWS CloudWatch or Splunk trying to find the server logs from the exact timestamp the test failed. This manual investigation takes hours.

In this tutorial, we will learn how to use **Log Aggregation** to automatically inject the backend server logs directly into our Pytest HTML reports!

---

## 1. The Goal: Unified Observability

When a test fails, our Pytest HTML report usually looks like this:

```text
FAIL: test_checkout.py::test_credit_card
Exception: TimeoutException waiting for element 'id=success'
Screenshot: checkout_fail.png
```

We want to add a brand new section to our report that looks like this:

```text
--- BACKEND SERVER LOGS ---
[10:42:01] INFO: Processing Payment for User admin
[10:42:03] ERROR: Stripe API Gateway Timeout (504)
[10:42:04] FATAL: Transaction Rolled Back.
```

If the QA Engineer sees this in their HTML report, they don't have to guess why the `TimeoutException` occurred. They know the Stripe API went down! They can instantly file a bug report with the exact root cause, skipping the manual investigation entirely.

---

## 2. Setting Up the Correlation ID

To query the backend logs for a specific test, the backend server needs to know *which* test triggered the request. 

We accomplish this using a **Correlation ID**. A Correlation ID is a unique string (usually a UUID) that we generate in our Python test and attach as an HTTP Header to every request we send to the backend.

```python
import uuid
import pytest
from selenium import webdriver
@pytest.fixture
def driver():
    # 1. Generate a unique Correlation ID for this specific test
    test_run_id = str(uuid.uuid4())
    # 2. Add it to Chrome's default headers (requires a proxy like Browsermob, or Chrome CDP)
    # For modern Selenium (v4+), we can use the CDP (Chrome DevTools Protocol)
    options = webdriver.ChromeOptions()
    driver = webdriver.Chrome(options=options)
    driver.execute_cdp_cmd('Network.enable', {})
    driver.execute_cdp_cmd('Network.setExtraHTTPHeaders', {
        'headers': {
            'X-Correlation-ID': test_run_id
        }
    })
    # We attach the ID to the driver object so the teardown function can access it
    driver.correlation_id = test_run_id
    yield driver
    driver.quit()
```

By injecting `X-Correlation-ID` into Chrome's DevTools Protocol, every single HTTP request Chrome makes (fetching HTML, making AJAX calls, loading images) will include our unique UUID!

---

## 3. Querying the Log Aggregator (Elasticsearch / Splunk)

If our backend server is configured properly, it will read the `X-Correlation-ID` header and attach it to every log message it prints.

When our Pytest test fails, we can use a `conftest.py` hook to dynamically query Elasticsearch for all logs containing that exact UUID!

```python
import requests
import pytest
def fetch_backend_logs(correlation_id):
    """
    Queries Elasticsearch for all server logs matching the Correlation ID.
    """
    es_url = "https://logs.mycodeyatra.com/_search"
    query = {
        "query": {
            "match": {
                "correlation_id": correlation_id
            }
        },
        "sort": [{"@timestamp": "asc"}]
    }
    response = requests.post(es_url, json=query)
    logs = response.json().get("hits", {}).get("hits", [])
    formatted_logs = "\n--- BACKEND SERVER LOGS ---\n"
    for log in logs:
        msg = log["_source"]["message"]
        time = log["_source"]["@timestamp"]
        formatted_logs += f"[{time}] {msg}\n"
    return formatted_logs
```

---

## 4. Injecting Logs into the Pytest Report

Finally, we use the `pytest_runtest_makereport` hook to intercept the failure, fetch the logs using our function, and inject them into the Pytest report!

```python
@pytest.hookimpl(tryfirst=True, hookwrapper=True)
def pytest_runtest_makereport(item, call):
    outcome = yield
    report = outcome.get_result()
    # If the test failed during the 'call' phase
    if report.when == "call" and report.failed:
        # Get the driver from the test item
        driver = item.funcargs.get("driver")
        if driver and hasattr(driver, "correlation_id"):
            # Fetch the logs from Elasticsearch!
            backend_logs = fetch_backend_logs(driver.correlation_id)
            # Append the logs directly to the Pytest terminal output and HTML report!
            report.sections.append(("Backend Logs", backend_logs))
```

## Conclusion

Log Aggregation is the ultimate bridge between Frontend Automation and Backend Observability. By injecting a `X-Correlation-ID` into Chrome via CDP, and dynamically querying Elasticsearch upon failure, you give your QA team X-Ray vision into the backend servers!

In the next tutorial, we will take a step back and define our overarching **Observability Strategy**—the philosophical framework that dictates how we monitor distributed microservices and asynchronous message queues using Pytest!
