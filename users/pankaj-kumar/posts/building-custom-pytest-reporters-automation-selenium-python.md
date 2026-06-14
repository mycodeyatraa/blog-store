---
title: Building Custom Pytest Reporters from Scratch
date: 22-Jun-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [python, selenium, pytest, custom-reporter, hooks, automation]
category: Reporting and Observability
categories: [Reporting and Observability, Python, Automation]
excerpt: >-
  Don't settle for off-the-shelf dashboards! Pop the hood of Pytest's hook architecture and learn how to write a custom reporter class to send automated Slack or Teams messages when your Selenium suite finishes.
readTime: 6 min read
---

# Building Custom Pytest Reporters from Scratch

Over the last few articles, we explored massive reporting frameworks like Allure and Test Analytics Dashboards. These are fantastic tools, but they are heavy. 

What if your company uses a proprietary internal messaging app (like Microsoft Teams, Slack, or a custom Discord bot), and the Engineering Director says: *"I don't want to log into a dashboard. I want a chat message sent to the #qa-alerts channel every time a test fails in the nightly run."*

There is no off-the-shelf plugin for your company's custom internal chat tool. You must build your own! In this article, we will "pop the hood" of Pytest and write a custom Pytest Reporter plugin from absolute scratch.

---

## 1. The Pytest Hook Architecture

Pytest is not a monolithic application; it is entirely hook-driven. Every time Pytest starts up, collects a test, runs a test, or finishes a test, it fires a "Hook".

If we want to build a custom reporter, we just need to write a Python class that listens to these specific hooks!

The three most important reporting hooks are:
1. `pytest_sessionstart(self, session)`: Fired once when the test run begins.
2. `pytest_runtest_logreport(self, report)`: Fired after every single test (Setup, Call, and Teardown).
3. `pytest_sessionfinish(self, session, exitstatus)`: Fired once when all tests are completely done.

---

## 2. Writing the Custom Reporter Class

Let's create a new file called `custom_reporter.py` in the root of our framework. 

We will create a class that tracks the number of passed and failed tests. When the entire test suite finishes, it will package these metrics into a custom JSON payload and send it via an HTTP POST request to our hypothetical company chat Webhook!

**custom_reporter.py**

```python
import requests
class CustomChatReporter:
    def __init__(self, webhook_url):
        self.webhook_url = webhook_url
        self.passed = 0
        self.failed = 0
        self.failed_tests = []
    # 1. Listen for individual test results
    def pytest_runtest_logreport(self, report):
        # We only care about the actual execution phase (not setup/teardown)
        if report.when == "call":
            if report.passed:
                self.passed += 1
            elif report.failed:
                self.failed += 1
                self.failed_tests.append(report.nodeid)
    # 2. Listen for the end of the entire test suite
    def pytest_sessionfinish(self, session, exitstatus):
        total = self.passed + self.failed
        # Calculate the success rate
        pass_rate = (self.passed / total * 100) if total > 0 else 0
        # Format our custom message
        message = f"🚀 **Nightly UI Automation Completed!** 🚀\n"
        message += f"Total Executed: {total}\n"
        message += f"✅ Passed: {self.passed}\n"
        message += f"❌ Failed: {self.failed}\n"
        message += f"📊 Pass Rate: {pass_rate:.1f}%\n"
        if self.failed > 0:
            message += "\n**Failing Tests:**\n"
            for test in self.failed_tests[:5]: # Show max 5 to avoid spam
                message += f"- {test}\n"
        # 3. Send the HTTP Request to the Chat Webhook!
        payload = {"text": message}
        print(f"\n[CustomReporter] Sending metrics to Webhook...")
        try:
            requests.post(self.webhook_url, json=payload)
            print("[CustomReporter] Success!")
        except Exception as e:
            print(f"[CustomReporter] Failed to send message: {e}")
```

---

## 3. Registering the Plugin with Pytest

We have written our logic, but Pytest does not know our class exists. We must register it during Pytest's initialization phase. 

We do this inside the global `conftest.py` file using the `pytest_configure` hook.

**tests/conftest.py**

```python
import pytest
from custom_reporter import CustomChatReporter
def pytest_configure(config):
    # Fetch the Webhook URL from the environment or a config file
    webhook_url = "https://hooks.mycompany.com/services/T000/B000/XXXX"
    # Instantiate our custom class
    my_reporter = CustomChatReporter(webhook_url)
    # Register it as a Pytest plugin!
    config.pluginmanager.register(my_reporter, "custom_chat_reporter")
```

### How it works:
1. You run `pytest tests/`.
2. Pytest executes `conftest.py` and registers `CustomChatReporter`.
3. Your Selenium tests begin executing.
4. After every test, Pytest calls `my_reporter.pytest_runtest_logreport()`, incrementing the `passed` and `failed` counters.
5. When the final test finishes, Pytest calls `my_reporter.pytest_sessionfinish()`.
6. Your class formats the data and executes the `requests.post()` call to your company's chat server!

## Conclusion

You are no longer limited to third-party tools! By understanding Pytest's Hook Architecture, you can build infinite custom integrations:
- Write a class that listens to `pytest_runtest_logreport`.
- Track metrics in memory.
- Use `pytest_sessionfinish` to trigger actions when the suite is complete.
- Register your class using `config.pluginmanager.register()`.

You can use this exact architecture to send SMS text messages via Twilio, automatically open Jira defect tickets, or trigger a physical red alarm siren in the QA department's office!

### 🎉 Phase 12 is Complete!
We have officially concluded Phase 12: Reporting and Observability. In our final chapter, **Phase 13: Advanced CI/CD and DevOps**, we will move our execution out of our local machine and into the Cloud using GitHub Actions and Docker!
