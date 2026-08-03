---
title: Decoupling User Actions: The Screenplay Pattern in Selenium Python
date: 12-Feb-2026
lastUpdated: 12-Feb-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: ["selenium", "python", "screenplay-pattern", "framework", "architecture", "solid"]
category: Framework Architecture
categories: ["Framework Architecture", "Selenium", "Python"]
excerpt: >-
  Transition from rigid Page Objects to user-centric Screenplay design! Master Actors, Abilities, Tasks, and Questions in Selenium Python for enterprise-grade maintainability.
readTime: 8 min read
---

# Decoupling User Actions: The Screenplay Pattern in Selenium Python

As test suites expand to thousands of test cases, even well-structured **Page Object Models (POM)** can suffer from maintenance strain. Page Object classes often balloon into massive 'God Objects' filled with hundreds of locators and methods.

The **Screenplay Pattern** addresses this by shifting focus from *pages* to *user intent*. Instead of asking *"What pages does this app have?"*, Screenplay asks *"Who is using the app, what can they do, and what are they trying to achieve?"*

---

## 1. Core Architectural Pillars of Screenplay

The Screenplay Pattern models test execution around four fundamental building blocks:

- **Actor**: Represents the user executing the test (e.g., `Pankaj`, `AdminUser`).
- **Ability**: Represents what the actor can do (e.g., `BrowseTheWeb` using Selenium `WebDriver`).
- **Task**: High-level workflow combining multiple interactions (e.g., `LoginToPortal`, `CheckoutCart`).
- **Question**: Query used to retrieve state and perform assertions (e.g., `TextOfElement`, `DashboardTitle`).

```
 +--------------+        can        +-------------------+
 |    Actor     | ----------------> | BrowseTheWeb (UI) |
 +--------------+                   +-------------------+
        |
    attempts_to
        |
        v
 +--------------+     performs      +-------------------+
 |  Task / Flow | ----------------> |  Target Locators  |
 +--------------+                   +-------------------+
```

---

## 2. Implementing the Screenplay Engine in Python

Below is a clean, extensible implementation of the core Screenplay engine in Python:

**screenplay/core.py**

```python
from typing import Any, List
 
class Actor:
    def __init__(self, name: str):
        self.name = name
        self.abilities = {}
 
    def can(self, ability: Any) -> "Actor":
        self.abilities[type(ability)] = ability
        return self
 
    def ability_to(self, ability_class: Any) -> Any:
        if ability_class not in self.abilities:
            raise KeyError(f"Actor {self.name} does not have the ability {ability_class.__name__}")
        return self.abilities[ability_class]
 
    def attempts_to(self, *tasks) -> None:
        for task in tasks:
            task.perform_as(self)
 
    def asks_for(self, question: Any) -> Any:
        return question.answered_by(self)
 
 
class BrowseTheWeb:
    def __init__(self, driver):
        self.driver = driver
 
    @staticmethod
    def as_actor(actor: Actor) -> "BrowseTheWeb":
        return actor.ability_to(BrowseTheWeb)
```

---

## 3. Defining Targets, Tasks, and Questions

Next, we decouple locators into lightweight `Target` objects and create reusable `Task` and `Question` classes:

**screenplay/tasks_and_questions.py**

```python
from selenium.webdriver.common.by import By
from screenplay.core import Actor, BrowseTheWeb
 
class Target:
    def __init__(self, name: str, locator: tuple):
        self.name = name
        self.locator = locator
 
    @staticmethod
    def the(name: str, locator: tuple) -> "Target":
        return Target(name, locator)
 
 
# Locators defined as independent targets
USERNAME_FIELD = Target.the("Username Input", (By.ID, "username"))
PASSWORD_FIELD = Target.the("Password Input", (By.ID, "password"))
LOGIN_BTN = Target.the("Login Button", (By.ID, "login-btn"))
WELCOME_MSG = Target.the("Welcome Heading", (By.TAG_NAME, "h1"))
 
 
class PerformLogin:
    def __init__(self, username: str, password: str):
        self.username = username
        self.password = password
 
    @staticmethod
    def with_credentials(username: str, password: str) -> "PerformLogin":
        return PerformLogin(username, password)
 
    def perform_as(self, actor: Actor) -> None:
        driver = BrowseTheWeb.as_actor(actor).driver
        driver.get("https://mycodeyatra.com/login")
        driver.find_element(*USERNAME_FIELD.locator).send_keys(self.username)
        driver.find_element(*PASSWORD_FIELD.locator).send_keys(self.password)
        driver.find_element(*LOGIN_BTN.locator).click()
 
 
class TextOfTarget:
    def __init__(self, target: Target):
        self.target = target
 
    @staticmethod
    def for_element(target: Target) -> "TextOfTarget":
        return TextOfTarget(target)
 
    def answered_by(self, actor: Actor) -> str:
        driver = BrowseTheWeb.as_actor(actor).driver
        return driver.find_element(*self.target.locator).text
```

---

## 4. Writing Executable Screenplay Tests

Here is how clean and readable tests become when written with Pytest and the Screenplay Pattern:

**tests/test_screenplay_login.py**

```python
import pytest
from selenium import webdriver
from screenplay.core import Actor, BrowseTheWeb
from screenplay.tasks_and_questions import PerformLogin, TextOfTarget, WELCOME_MSG
 
@pytest.fixture
def driver():
    driver = webdriver.Chrome()
    driver.implicitly_wait(10)
    yield driver
    driver.quit()
 
def test_user_can_login_successfully(driver):
    # 1. Initialize Actor and give Ability
    pankaj = Actor("Pankaj").can(BrowseTheWeb(driver))
 
    # 2. Actor attempts high-level Task
    pankaj.attempts_to(
        PerformLogin.with_credentials("standard_user", "secret_sauce")
    )
 
    # 3. Actor asks Question to assert outcome
    welcome_text = pankaj.asks_for(TextOfTarget.for_element(WELCOME_MSG))
    assert "Welcome" in welcome_text
```

---

## 5. Screenplay vs. Page Object Model Comparison

| Aspect | Page Object Model (POM) | Screenplay Pattern |
| :--- | :--- | :--- |
| **Focus** | Page URLs and screen elements | User roles, abilities, and goals |
| **Class Growth** | High (God objects with 500+ lines) | Low (Single-responsibility task classes) |
| **Reusability** | Page-level methods | Granular tasks and questions |
| **SOLID Compliance** | Often violates Single Responsibility Principle | Strictly enforces Single Responsibility Principle |

---

## Key Takeaways & Best Practices

1. **Keep Tasks Focused**: Each task class should perform one clear user intent.
2. **Reuse Questions**: Build a standard library of Questions (`TextOfTarget`, `VisibilityOfTarget`, `AttributeOfTarget`).
3. **Decouple Locators**: Store `Target` locators in separate modular components to simplify UI refactoring.
