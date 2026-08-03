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
tags: ["selenium", "python", "page-object-model", "pom", "framework", "architecture"]
category: Framework Architecture
categories: ["Framework Architecture", "Selenium", "Python"]
excerpt: >-
  Transition from Page Objects to actors, abilities, and tasks. Learn how to write cleaner, more maintainable Selenium Python tests with the Screenplay Pattern.
readTime: 4 min read
---

# Decoupling User Actions: The Screenplay Pattern in Selenium Python

The Screenplay Pattern is a user-centric approach to test automation that emphasizes actors, their abilities, and the tasks they perform.

---

## 1. Actor, Ability, and Task Model

Instead of modeling tests around pages, we structure them around actors:
- **Actor:** Who is executing the action (e.g., Pankaj).
- **Ability:** What the actor can do (e.g., Browse the Web using Selenium WebDriver).
- **Task:** High-level actions the actor performs (e.g., Add Item to Cart).

---

## 2. Implementing Screenplay in Python

Here is a clean implementation of the Screenplay Pattern structure:

```python
class Actor:
    def __init__(self, name):
        self.name = name
        self.abilities = {}
# ---
    def can(self, ability):
        self.abilities[type(ability)] = ability
        return self
# ---
    def attempts_to(self, *tasks):
        for task in tasks:
            task.perform_as(self)
# ---
class BrowseTheWeb:
    def __init__(self, driver):
        self.driver = driver
# ---
    @staticmethod
    def as_actor(actor):
        return actor.abilities[BrowseTheWeb]
```

This structural shift makes your automation code decoupled, highly reusable, and readable.
