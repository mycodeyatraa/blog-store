---
title: Filtering Execution: Using Tags in pytest-bdd
date: 18-Mar-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [python, selenium, bdd, pytest-bdd, tags, markers, test-execution]
category: Selenium Python
categories: [Selenium Python, Behavior Driven Development]
excerpt: >-
  Don't run your entire 2,000 test suite for a minor bug fix. Learn how to use Gherkin tags and pytest markers to strategically execute specific smoke and regression suites.
readTime: 4 min read
---

# Filtering Execution: Using Tags in pytest-bdd

As your Behavior Driven Development (BDD) framework grows, you will eventually have hundreds of `.feature` files containing thousands of scenarios. 

When a developer submits a small pull request fixing a minor UI bug, you do not want to run the entire suite of 2,000 tests. You only want to run the fast "Smoke" tests. Alternatively, before a major production release, you might want to run a specific "Regression" suite.

In this tutorial, we will learn how to use **Tags** in Gherkin and `pytest-bdd` to strategically filter and organize test execution!

---

## 1. Tagging in Gherkin

A tag is simply an annotation starting with an `@` symbol placed above a `Feature` or a `Scenario`.

Here is an example `login.feature` file:

```gherkin
@login @regression
Feature: User Authentication
  @smoke @critical
  Scenario: Successful login with valid credentials
    Given I navigate to the login page
    When I enter valid credentials
    Then I should see the dashboard
  @edge-case
  Scenario: Login with an extremely long password
    Given I navigate to the login page
    When I enter a 500-character password
    Then I should see a validation error
```

### Tag Inheritance
If you place a tag above the `Feature` (like `@login`), **every single scenario inside that file automatically inherits that tag!** 

In the example above, the first scenario has the tags: `@login, @regression, @smoke, @critical`.

---

## 2. Executing Tags via Pytest CLI

The true power of tags comes during CLI execution. `pytest` provides a built-in flag `-m` (which stands for markers/tags) to filter tests!

### Running a Single Tag
To run only the scenarios tagged with `@smoke`:

```bash
pytest tests/ -m smoke
```

### Running Multiple Tags (AND Logic)
To run scenarios that have BOTH `@login` AND `@smoke`:

```bash
pytest tests/ -m "login and smoke"
```

### Running Multiple Tags (OR Logic)
To run scenarios that have EITHER `@smoke` OR `@critical`:

```bash
pytest tests/ -m "smoke or critical"
```

### Excluding Tags (NOT Logic)
To run all tests EXCEPT the slow regression ones:

```bash
pytest tests/ -m "not regression"
```

---

## 3. Registering Tags in `pytest.ini`

If you start running the commands above, you might notice a bunch of yellow warning messages in your terminal:
`PytestUnknownMarkWarning: Unknown pytest mark @smoke`

By default, Pytest requires you to formally "register" your custom tags. This prevents typos (like accidentally typing `@smkoe` instead of `@smoke`).

To register your BDD tags, create or update a `pytest.ini` file in the root of your project:

```ini
[pytest]
markers =
    smoke: Quick tests intended to verify core functionality
    regression: Slower, comprehensive tests
    critical: Tests that assert P0 level business requirements
    login: Authentication related tests
    edge-case: Strange, non-standard user flows
```

Once this file exists, Pytest will no longer throw warnings, and it will throw a fatal error if a developer accidentally misspells a tag in a `.feature` file!

---

## 4. Hooking into Tags Programmatically

Tags are not just for CLI filtering. You can actually read the tags programmatically inside your `conftest.py` hooks to change test behavior dynamically!

For example, imagine you want to spin up a mobile-emulation browser ONLY if the scenario has a `@mobile` tag:

```python
def pytest_bdd_before_scenario(request, feature, scenario):
    # scenario.tags is a list of all tags attached to the scenario!
    if "mobile" in scenario.tags:
        print("\n📱 Booting up Mobile Emulator...")
        # (Inject mobile specific WebDriver options here)
    else:
        print("\n💻 Booting up Standard Desktop Browser...")
```

This dynamic approach allows you to completely alter the infrastructure state based on simple English annotations in your Gherkin files!

## Conclusion

Tags are the primary mechanism for organizing a massive Enterprise test suite. By strategically applying `@smoke`, `@regression`, and feature-specific tags, you give your DevOps pipelines ultimate control over what gets executed and when.

In the final tutorial for Phase 10, we will zoom out and discuss the **Behave Framework**—an alternative to `pytest-bdd`—and compare their architectural philosophies!
