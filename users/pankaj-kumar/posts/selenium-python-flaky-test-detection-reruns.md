---
title: Identifying the Culprits: Flaky Test Detection in Selenium
date: 02-Apr-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [python, selenium, flaky-tests, pytest, retries, test-automation, debugging]
category: Selenium Python
categories: [Selenium Python, Failure Analysis & Analytics]
excerpt: >-
  Flakiness destroys trust. Learn how to use pytest-rerunfailures to retry timeouts, and how to use Pytest hooks to automatically build a naughty list of tests that flap.
readTime: 5 min read
---

# Identifying the Culprits: Flaky Test Detection in Selenium

If you ask any Automation Engineer what their biggest nightmare is, the answer is unanimous: **Flakiness**.

A flaky test is a test that fails on Monday, passes on Tuesday, and fails again on Wednesday—without any code changing. Flaky tests destroy trust. If your developers do not trust the CI/CD pipeline, they will simply ignore the red `X` and merge their code anyway, completely defeating the purpose of automation!

In this tutorial, we will learn how to combat flakiness using the `pytest-rerunfailures` plugin, and how to programmatically track and quarantine tests that consistently flap.

---

## 1. Automated Retries

The easiest way to silence a flaky test that occasionally times out due to server load is to simply retry it. 

We can install a fantastic Pytest plugin that does this automatically:

```bash
pip install pytest-rerunfailures
```

Once installed, you can pass the `--reruns` flag to your execution command:

```bash
pytest tests/ --reruns 2 --reruns-delay 5
```

This command tells Pytest: *"If a test fails, wait 5 seconds and try again. Do this up to 2 times before officially marking it as FAILED."*

---

## 2. The Danger of Retries (Masking Flakiness)

While `--reruns` is incredibly useful, it is also highly dangerous. 

If Test A fails on the first attempt but passes on the second attempt, Pytest will mark the entire suite as **PASSED**. Over time, your team might have 200 tests that secretly fail on the first attempt every single day. You have successfully masked the flakiness, but you haven't fixed the underlying synchronization issues!

To truly fix flakiness, we need to **Detect** it.

---

## 3. Detecting Flakiness via Pytest Hooks

Just like we did in the previous tutorial, we can hook into Pytest's lifecycle. `pytest-rerunfailures` adds a custom outcome called `RERUN` to the report object.

We can intercept this outcome to build a "Naughty List" of flaky tests!

```python
# conftest.py
import pytest
# A global dictionary to track which tests had to be rerun
flaky_tests_registry = {}
@pytest.hookimpl(tryfirst=True, hookwrapper=True)
def pytest_runtest_makereport(item, call):
    outcome = yield
    report = outcome.get_result()
    # Check if the plugin marked this execution as a "RERUN"
    if report.outcome == "rerun":
        test_name = item.nodeid
        # Increment the counter for this specific test
        if test_name in flaky_tests_registry:
            flaky_tests_registry[test_name] += 1
        else:
            flaky_tests_registry[test_name] = 1
```

---

## 4. Emitting the Flaky Report

Now, let's output our findings at the end of the execution. We want to publicly shame any test that had to be retried!

```python
def pytest_sessionfinish(session, exitstatus):
    """
    Runs once at the very end of the test suite.
    """
    if not flaky_tests_registry:
        return
    print("\n" + "="*50)
    print(" 🚨 FLAKY TEST DETECTION REPORT 🚨 ")
    print("="*50)
    print("The following tests passed, but required retries:")
    # Sort the dictionary by the highest number of retries!
    sorted_flaky = sorted(flaky_tests_registry.items(), key=lambda x: x[1], reverse=True)
    for test_name, retry_count in sorted_flaky:
        print(f"[{retry_count} Retries]: {test_name}")
    print("="*50 + "\n")
```

When your CI pipeline finishes, it might say the suite Passed, but the logs will clearly show:

```text
==================================================
 🚨 FLAKY TEST DETECTION REPORT 🚨 
==================================================
The following tests passed, but required retries:
[2 Retries]: tests/test_checkout.py::test_credit_card_submission
[1 Retries]: tests/test_login.py::test_sso_redirect
==================================================
```

---

## 5. The Enterprise Strategy: Quarantine

In massive Enterprise teams (like Netflix or Google), detecting a flaky test is only step one. 

Step two is **Quarantine**. If a specific test appears on the Flaky Test Detection Report more than 5 times in a week, the Automation Architect will actively remove the `@smoke` tag and replace it with a `@quarantine` tag.

Quarantined tests are entirely excluded from the main CI/CD pipeline blocking execution. Instead, they run in a separate "nightly" pipeline where developers can investigate their synchronization issues without holding up production releases!

## Conclusion

Flakiness is inevitable, but masking it with endless retries is a catastrophic mistake. By combining `pytest-rerunfailures` with Pytest lifecycle hooks, you can automatically detect, track, and ultimately quarantine tests that drain your team's confidence!

In the next tutorial, we will zoom out from the terminal entirely and explore **Test Analytics**—learning how to push our failure metrics into time-series databases to track the health of our framework over months and years!
