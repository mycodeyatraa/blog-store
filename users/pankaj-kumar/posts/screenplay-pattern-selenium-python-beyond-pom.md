---
title: The Screenplay Pattern: Beyond the Page Object Model
date: 08-Jan-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [selenium, python, screenplay-pattern, pom, design-pattern, enterprise]
category: Framework Architecture
categories: [Framework Architecture, Selenium, Python]
excerpt: >-
  Is your Page Object Model becoming a bloated nightmare? Discover the Screenplay Pattern: an Actor-centric, highly composable architecture designed for massive enterprise UI test suites.
readTime: 6 min read
---

# The Screenplay Pattern: Beyond the Page Object Model

The Page Object Model (POM) is the undisputed king of UI automation architecture. However, in massive enterprise applications with hundreds of interconnected pages, POM can start to show its flaws.

Have you ever opened a `dashboard_page.py` file and found 2,000 lines of code and 150 different methods? This violates the **Single Responsibility Principle**. When a class becomes a "God Object" that knows everything and does everything, it becomes a nightmare to maintain.

Enter the **Screenplay Pattern**.

The Screenplay Pattern shifts the focus away from *Pages* (the UI structure) and places it on *Actors* (the users and their goals). 

---

## 1. The Core Concepts of Screenplay

The Screenplay Pattern reads like an English sentence: 
**"An Actor uses an Ability to perform Tasks, which are made up of Actions, to answer Questions about the system."**

Let's break down this vocabulary:
1. **Actor**: The entity performing the test (e.g., "Admin", "Customer").
2. **Ability**: What the Actor can do (e.g., "Browse the Web using Selenium").
3. **Tasks**: High-level business goals (e.g., "Login", "Purchase Item").
4. **Actions**: Low-level UI interactions (e.g., "Click", "Type").
5. **Questions**: Assertions about the state of the UI (e.g., "Is the welcome message visible?").

---

## 2. Setting up the Screenplay Architecture

Instead of organizing our framework by "Pages", we organize it by the concepts above.

Let's start by defining our low-level `Actions` and `Questions`. Notice how small and focused these classes are!

**actions.py**

```python
class EnterText:
    def __init__(self, text, locator):
        self.text = text
        self.locator = locator
    def perform_as(self, actor):
        driver = actor.ability.driver
        driver.find_element(*self.locator).send_keys(self.text)
class Click:
    def __init__(self, locator):
        self.locator = locator
    def perform_as(self, actor):
        driver = actor.ability.driver
        driver.find_element(*self.locator).click()
```

**questions.py**

```python
class TextOf:
    def __init__(self, locator):
        self.locator = locator
    def answered_by(self, actor):
        driver = actor.ability.driver
        return driver.find_element(*self.locator).text
```

---

## 3. Composing Tasks

Now, we compose these small, reusable Actions into a high-level **Task**. A Task is simply a collection of Actions.

**tasks.py**

```python
from actions import EnterText, Click
class LoginTask:
    def __init__(self, username, password):
        self.username = username
        self.password = password
    # We still store Locators, but we separate them from the Actions!
    USER_INPUT = ("id", "username")
    PASS_INPUT = ("id", "password")
    LOGIN_BTN = ("id", "login-btn")
    def perform_as(self, actor):
        # A Task delegates work to the low-level Actions
        actor.attempts_to(
            EnterText(self.username, self.USER_INPUT),
            EnterText(self.password, self.PASS_INPUT),
            Click(self.LOGIN_BTN)
        )
```

---

## 4. The Actor and Ability

Finally, we need the "Glue" that holds this all together. 
The `Actor` is just a class that holds an `Ability` (the Selenium WebDriver) and executes Tasks.

**actor.py**

```python
class BrowseTheWeb:
    def __init__(self, driver):
        self.driver = driver
class Actor:
    def __init__(self, name):
        self.name = name
        self.ability = None
    def can(self, ability):
        self.ability = ability
        return self
    def attempts_to(self, *tasks):
        for task in tasks:
            task.perform_as(self)
    def asks(self, question):
        return question.answered_by(self)
```

---

## 5. Writing the Screenplay Test

Let's look at the final test file. Notice how incredibly expressive and readable the Screenplay Pattern is. It completely hides the Selenium WebDriver implementation details from the test!

**test_screenplay.py**

```python
import pytest
from selenium import webdriver
from actor import Actor, BrowseTheWeb
from tasks import LoginTask
from questions import TextOf
@pytest.fixture
def driver():
    driver = webdriver.Chrome()
    driver.get("https://mycodeyatra.com/login")
    yield driver
    driver.quit()
def test_screenplay_login(driver):
    # 1. Define the Actor and give them the Ability to use Selenium
    james = Actor("James").can(BrowseTheWeb(driver))
    # 2. The Actor attempts a high-level Task
    james.attempts_to(
        LoginTask("admin", "secure_password")
    )
    # 3. The Actor asks a Question to verify the outcome
    WELCOME_MSG = ("id", "toast")
    assert james.asks(TextOf(WELCOME_MSG)) == "Welcome back, James!"
```

---

## 6. Execution Output

When executed via PyTest, the underlying Selenium actions fire just like a normal POM framework, resulting in a successful test execution.

```text
============================= test session starts ==============================
platform win32 -- Python 3.12.0, pytest-8.0.0
rootdir: C:\Automation\mycodeyatra-tests
collected 1 item
test_screenplay.py::test_screenplay_login PASSED                         [100%]
============================== 1 passed in 4.95s ===============================
```

## Conclusion

The Screenplay Pattern fixes the "God Object" problem of POM.
- You no longer have massive Page Classes. 
- You have dozens of tiny, highly reusable `Task` and `Action` classes.
- `EnterText` can be used on the Login Page, the Registration Page, and the Checkout Page!

While Screenplay is incredibly powerful for massive enterprise suites, the initial architecture setup can be complex. For small to medium projects, POM remains the go-to standard.

In our next article, we will move away from design patterns and look at the engine powering our entire suite: **PyTest Fixtures and Hooks!**
