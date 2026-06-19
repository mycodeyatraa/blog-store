---
title: The Bigger Picture: Test Analytics in Python Automation
date: 04-Apr-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [python, selenium, test-analytics, elasticsearch, grafana, metrics, reporting]
category: Selenium Python
categories: [Selenium Python, Failure Analysis & Analytics]
excerpt: >-
  Stop relying on daily HTML reports. Learn how to extract pytest execution metrics and push them to Elasticsearch to build beautiful, historical Grafana dashboards tracking suite health over time.
readTime: 5 min read
---

# The Bigger Picture: Test Analytics in Python Automation

If you run a suite of 2,000 Selenium tests every single night, you are generating a massive amount of data. 

- Which tests are failing the most this month?
- How much longer did the suite take to run today compared to last week?
- Which specific developer's commit caused the sudden spike in `TimeoutExceptions`?

A simple HTML report cannot answer these questions, because HTML reports are ephemeral—they only show the results of a *single* run. 

To answer these questions, we must transition from simple "Reporting" to **Test Analytics**. In this tutorial, we will learn how to extract metadata from Pytest and push it into an external database to build beautiful historical dashboards!

---

## 1. What are Test Analytics?

**Test Reporting** answers the question: *"Did the suite pass today?"*
**Test Analytics** answers the question: *"Is the overall health of our automation framework improving or degrading over time?"*

To build a Test Analytics pipeline, you need three components:
1. **The Extractor:** A Pytest hook that captures the duration and status of every test.
2. **The Database:** A time-series database (like InfluxDB, PostgreSQL, or Elasticsearch) to store the historical data.
3. **The Dashboard:** A visualization tool (like Grafana or Kibana) to graph the trends.

---

## 2. Extracting Data with Pytest Hooks

We need to capture the exact duration (in milliseconds) and the final outcome (Pass/Fail) of every single test.

We can achieve this using the `pytest_runtest_makereport` hook in our `conftest.py`:

```python
import pytest
import time
import requests
# A global list to store the results of the current run
run_metrics = []
@pytest.hookimpl(tryfirst=True, hookwrapper=True)
def pytest_runtest_makereport(item, call):
    outcome = yield
    report = outcome.get_result()
    # We only want to record metrics when the test ACTUALLY finishes executing
    if report.when == "call":
        metric = {
            "test_name": item.nodeid,
            "status": report.outcome,
            "duration_ms": int(report.duration * 1000),
            "timestamp": int(time.time() * 1000)
        }
        run_metrics.append(metric)
```

---

## 3. Pushing to a Time-Series Database

Once the entire test suite finishes, we have an array containing the exact metadata for every test. Now, we just need to push it to our database!

Let's assume our DevOps team set up an **Elasticsearch** cluster for us. We can use the `pytest_sessionfinish` hook to perform a bulk `POST` request.

```python
def pytest_sessionfinish(session, exitstatus):
    """
    Runs once when the entire test suite finishes.
    """
    if not run_metrics:
        return
    print(f"\n📊 Pushing {len(run_metrics)} metrics to Elasticsearch...")
    # Example Elasticsearch bulk insert
    es_url = "https://metrics.mycodeyatra.com/test-runs/_bulk"
    bulk_data = ""
    for metric in run_metrics:
        # Elasticsearch bulk API requires a metadata line followed by the JSON line
        bulk_data += '{"index": {}}\n'
        bulk_data += str(metric).replace("'", '"') + "\n"
    try:
        response = requests.post(
            es_url, 
            data=bulk_data, 
            headers={"Content-Type": "application/x-ndjson"}
        )
        if response.status_code == 200:
            print("✅ Metrics successfully uploaded!")
        else:
            print(f"❌ Failed to push metrics: {response.status_code}")
    except Exception as e:
        print(f"⚠️ Could not reach metrics server: {e}")
```

---

## 4. Visualizing the Data (Grafana/Kibana)

Once your metrics are flowing into Elasticsearch or InfluxDB, the magic happens in your visualization dashboard.

You can instantly create graphs that track:
- **Suite Duration Over Time:** A line graph showing if your suite is getting slower week over week. (A sudden spike indicates a developer might have added a massive `time.sleep()` somewhere!)
- **Top 10 Flakiest Tests:** A bar chart showing the tests with the highest fail rates over the last 30 days.
- **Pass Rate by Feature:** A pie chart showing which specific modules of your application (e.g., Checkout vs Profile) are the most unstable.

## Conclusion

Transitioning to Test Analytics is the hallmark of a mature QA team. Instead of reacting to daily failures, you gain the ability to proactively identify degrading performance, spot intermittent synchronization issues before they become permanent failures, and definitively prove the ROI of your automation framework to senior management!

In the final tutorial of Phase 12, we will bring everything together and learn how to write our very own **Custom Reporters** in Pytest, allowing us to output our results to Slack, Microsoft Teams, or custom HTML templates!
