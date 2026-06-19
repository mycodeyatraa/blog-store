---
title: Beyond HTML: Building Custom Reporters in Selenium Python
date: 06-Apr-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [python, selenium, custom-reporters, slack, webhook, pytest, test-automation]
category: Selenium Python
categories: [Selenium Python, Failure Analysis & Analytics]
excerpt: >-
  Ditch the boring HTML reports. Learn how to use pytest_terminal_summary to build custom Python reporters that broadcast test suite results directly to Slack or Microsoft Teams via webhooks.
readTime: 4 min read
---

# Beyond HTML: Building Custom Reporters in Selenium Python

We've learned how to generate beautiful standard reports using `pytest-html` and Allure. We've learned how to push metrics to Elasticsearch for historical analytics.

But what if your Engineering Director comes to you and says: *"I don't want to log into Grafana, and I don't want to open an HTML file. I just want a summary message sent to our #qa-automation Slack channel every morning at 8:00 AM."*

In this final tutorial of Phase 12, we will learn how to build a **Custom Pytest Reporter**. We will aggregate our test results entirely in memory, and use a Python webhook to instantly broadcast our results to external systems like Slack or Microsoft Teams!

---

## 1. The Power of `pytest_terminal_summary`

Pytest provides a magnificent hook called `pytest_terminal_summary`. This hook fires at the absolute end of the test run, right before Pytest prints its final output to the terminal.

It provides access to a `terminalreporter` object, which contains the final tally of every test status (passed, failed, skipped, error).

Let's open our `conftest.py` and hook into it:

```python
import pytest
import requests
def pytest_terminal_summary(terminalreporter, exitstatus, config):
    # Extract the exact counts from the terminal reporter!
    passed = len(terminalreporter.stats.get("passed", []))
    failed = len(terminalreporter.stats.get("failed", []))
    skipped = len(terminalreporter.stats.get("skipped", []))
    error = len(terminalreporter.stats.get("error", []))
    total = passed + failed + skipped + error
    # Calculate a pass percentage
    if total > 0:
        pass_rate = round((passed / total) * 100, 2)
    else:
        pass_rate = 0.0
    print(f"\n📊 Final Pass Rate: {pass_rate}%")
```

---

## 2. Formatting the Slack Webhook Payload

Now that we have the exact data, we need to format it into a message that Slack understands.

Slack allows you to create "Incoming Webhooks"—unique URLs that accept HTTP `POST` requests containing JSON payloads. You can use Slack's "Block Kit" formatting to make the message look beautiful with colored sidebars and bold text.

Let's write a function to construct the payload:

```python
def send_slack_notification(passed, failed, skipped, total, pass_rate):
    webhook_url = "https://hooks.slack.com/services/T0000/B0000/XXXX"
    # Choose a color: Green if perfect, Red if anything failed
    color = "#36a64f" if failed == 0 else "#ff0000"
    # Construct the Slack Block Kit Payload
    payload = {
        "attachments": [
            {
                "color": color,
                "title": "🧪 Nightly Automation Run Complete",
                "text": f"*Pass Rate:* {pass_rate}%\n*Total Tests:* {total}",
                "fields": [
                    {
                        "title": "✅ Passed",
                        "value": str(passed),
                        "short": True
                    },
                    {
                        "title": "❌ Failed",
                        "value": str(failed),
                        "short": True
                    },
                    {
                        "title": "⏭️ Skipped",
                        "value": str(skipped),
                        "short": True
                    }
                ],
                "footer": "Selenium Python Automation Framework"
            }
        ]
    }
    # Fire the request!
    try:
        response = requests.post(webhook_url, json=payload)
        if response.status_code == 200:
            print("🚀 Slack notification sent successfully!")
        else:
            print(f"⚠️ Failed to send Slack message: {response.text}")
    except Exception as e:
        print(f"⚠️ Could not reach Slack: {e}")
```

---

## 3. Assembling the Custom Reporter

Let's put it all together inside our `conftest.py`. We will only send the Slack message if we are running in our CI/CD environment (we don't want to spam the Slack channel every time a developer runs a test locally on their laptop!).

```python
import os
def pytest_terminal_summary(terminalreporter, exitstatus, config):
    passed = len(terminalreporter.stats.get("passed", []))
    failed = len(terminalreporter.stats.get("failed", []))
    skipped = len(terminalreporter.stats.get("skipped", []))
    total = passed + failed + skipped
    if total == 0:
        return
    pass_rate = round((passed / total) * 100, 2)
    # Check if we are running in CI (e.g., GitHub Actions or Jenkins)
    is_ci = os.getenv("CI", "false").lower() == "true"
    if is_ci:
        send_slack_notification(passed, failed, skipped, total, pass_rate)
    else:
        print("\n[Local Run] Skipping Slack Notification.")
```

## Conclusion: Phase 12 Complete!

By mastering the `pytest_terminal_summary` hook, you have unbound your framework from generic HTML files. You can now build Custom Reporters that send data to Slack, Microsoft Teams, Jira, or even a custom internal dashboard!

This officially concludes **Phase 12: Failure Analysis & Analytics**. We have learned how to categorize errors, detect flaky tests, push to time-series databases, and build custom webhooks!

In the final phase of this curriculum, **Phase 13: CI/CD Pipeline**, we will take everything we have built and learn how to run it inside Docker containers via GitHub Actions!
