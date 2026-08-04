---
title: Screenplay Pattern - Playwright Java Design Patterns
date: 27-Jan-2026
lastUpdated: 27-Jan-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [playwright, java, junit5, automation, design-patterns, mycodeyatra]
category: Playwright Java Design Patterns
categories: [Playwright Java Design Patterns, Playwright Java, Test Automation]
excerpt: >-
  Implement SOLID principles in QA using the Actor-Task-Ability Screenplay Pattern in Playwright Java.
readTime: 9 min read
---

# Screenplay Pattern - Playwright Java Design Patterns

The Screenplay Pattern is a user-centric design pattern that replaces traditional Page Objects with **Actors**, **Abilities**, **Tasks**, and **Questions**.

By applying SOLID principles, Screenplay ensures that test steps read like acceptance criteria while keeping interactions completely decoupled from test runners.

---

## 1. Screenplay Core Concepts

- **Actor**: The user performing actions (e.g. `Actor.named("Pankaj")`).
- **Ability**: What the actor can do (e.g. `BrowseTheWeb.with(page)`).
- **Task**: High-level user goal (e.g. `SubmitRegistrationForm.withDetails(...)`).
- **Question**: Assertions checking state (e.g. `TheSuccessBanner.value()`).

---

## 2. Screenplay Code Architecture

```java
// 1. Actor & Ability Implementation
public class Actor {
    private final String name;
    private Page page;
 
    public Actor(String name) {
        this.name = name;
    }
 
    public static Actor named(String name) {
        return new Actor(name);
    }
 
    public Actor can(Page page) {
        this.page = page;
        return this;
    }
 
    public void attemptsTo(Runnable task) {
        task.run();
    }
}
 
// 2. Executing Screenplay Pattern Test
Actor pankaj = Actor.named("Pankaj").can(page);
pankaj.attemptsTo(() -> {
    page.navigate("https://practice.mycodeyatra.com/form-practice");
    page.fill("#username", "Pankaj Kumar");
    page.click("#submit-btn");
});
```

---

## 3. Production Page Object (`src/main/java/com/mycodeyatra/pages/design/ScreenplayActorPage.java`)

```java
package com.mycodeyatra.pages.design;
 
import com.microsoft.playwright.Page;
 
public class ScreenplayActorPage {
    private final Page page;
 
    public ScreenplayActorPage(Page page) {
        this.page = page;
    }
 
    public void performAction() {
        page.navigate("https://practice.mycodeyatra.com/sandbox");
    }
}
```

---

## 4. Key Takeaways

1. **Reflect User Intent**: Name tasks based on business capabilities rather than UI click mechanics.
2. **High Composability**: Re-use granular task blocks across multiple test scenarios.

