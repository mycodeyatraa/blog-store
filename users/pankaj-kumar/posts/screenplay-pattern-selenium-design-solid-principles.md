---
title: Screenplay Pattern in Selenium: Refactoring POM with SOLID Principles
date: 26-Jul-2024
lastUpdated: 11-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [selenium, java, screenplay-pattern, design-patterns, solid-principles, clean-code]
category: UI Automation
categories: [UI Automation, Selenium, Java]
excerpt: >-
  Master the Screenplay Pattern in Selenium WebDriver. Learn how to refactor bloated Page Objects (POM) into clean, decoupled Actors, Abilities, Tasks, and Questions in Java.
readTime: 6 min read
---

# Screenplay Pattern in Selenium: Refactoring POM with SOLID Principles

> 📅 **Last Updated:** 11-Jun-2026

As automated test suites grow in size, traditional Page Objects (POM) often become bloated. Page classes end up carrying double responsibilities: maintaining web element locators and defining high-level workflow interactions. This violates the **Single Responsibility Principle (SRP)**: any small change to the UI or business workflow requires modifying the exact same page class.

The **Screenplay Pattern** is a user-centric, actor-based design pattern that refactors POM by separating actors, their abilities, business tasks, and assertions into independent, decoupled classes. It strictly enforces the SOLID design principles in test automation.

---

## 🧭 Screenplay Pattern Architecture

The Screenplay architecture separates concerns into four distinct components:
1. **Actor**: Who is performing the test (e.g., "James").
2. **Abilities**: What the Actor can do (e.g., BrowseTheWeb using a WebDriver).
3. **Tasks/Interactions**: Actions the Actor performs (e.g., NavigateTask, LoginTask).
4. **Questions**: Queries the Actor makes to check system state (e.g., ProfileHeaderQuestion).

The diagram below maps how these decoupled components interact:

![diagram_1](https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/posts/screenplay-pattern-selenium-design-solid-principles/images/diagram_1.png)

---

## 🛠️ Step-by-Step Code Walkthrough

We implement the core Screenplay engines under src/main/java/com/mycodeyatra/screenplay/ and the validation test under src/test/java/com/mycodeyatra/tests/.

### 1. Core Interfaces & Engines
First, we define the base contracts for Ability, Performable (Tasks), and Question types:

#### Ability (Ability.java)

```java
package com.mycodeyatra.screenplay;

public interface Ability {
}
```

#### Performable (Performable.java)

```java
package com.mycodeyatra.screenplay;

public interface Performable {
    void performAs(Actor actor);
}
```

#### Question (Question.java)

```java
package com.mycodeyatra.screenplay;

public interface Question<T> {
    T answeredBy(Actor actor);
}
```

#### BrowseTheWeb Ability (BrowseTheWeb.java)
This class encapsulates the Selenium WebDriver instance, giving the Actor access to browser actions.

```java
package com.mycodeyatra.screenplay;

import org.openqa.selenium.WebDriver;

public class BrowseTheWeb implements Ability {
    private final WebDriver driver;

    private BrowseTheWeb(WebDriver driver) {
        this.driver = driver;
    }

    public static BrowseTheWeb with(WebDriver driver) {
        return new BrowseTheWeb(driver);
    }

    public WebDriver getDriver() {
        return driver;
    }
}
```

#### The Actor (Actor.java)
The actor maintains state, handles capabilities (abilities), executes tasks, and evaluates assertions (questions).

```java
package com.mycodeyatra.screenplay;

import java.util.HashMap;
import java.util.Map;

public class Actor {
    private final String name;
    private final Map<Class<? extends Ability>, Ability> abilities = new HashMap<>();

    private Actor(String name) {
        this.name = name;
    }

    public static Actor named(String name) {
        return new Actor(name);
    }

    public <T extends Ability> Actor can(T ability) {
        abilities.put(ability.getClass(), ability);
        return this;
    }

    @SuppressWarnings("unchecked")
    public <T extends Ability> T usingAbilityTo(Class<T> abilityClass) {
        T ability = (T) abilities.get(abilityClass);
        if (ability == null) {
            throw new IllegalArgumentException("Actor " + name + " does not have the ability: " + abilityClass.getSimpleName());
        }
        return ability;
    }

    public void attemptsTo(Performable... tasks) {
        for (Performable task : tasks) {
            task.performAs(this);
        }
    }

    public <T> T asksFor(Question<T> question) {
        return question.answeredBy(this);
    }

    public String getName() {
        return name;
    }
}
```

### 2. Custom Screenplay Tasks
Next, we define granular, reusable tasks. Notice how these tasks do not carry test assertions; they only drive actions:

#### NavigateTask (NavigateTask.java)

```java
package com.mycodeyatra.screenplay.tasks;

import com.mycodeyatra.screenplay.Actor;
import com.mycodeyatra.screenplay.BrowseTheWeb;
import com.mycodeyatra.screenplay.Performable;
import org.openqa.selenium.WebDriver;

public class NavigateTask implements Performable {
    private final String url;

    private NavigateTask(String url) {
        this.url = url;
    }

    public static NavigateTask to(String url) {
        return new NavigateTask(url);
    }

    @Override
    public void performAs(Actor actor) {
        WebDriver driver = actor.usingAbilityTo(BrowseTheWeb.class).getDriver();
        System.out.println("NavigateTask: " + actor.getName() + " is navigating to: " + url);
        driver.get(url);
    }
}
```

#### LoginTask (LoginTask.java)

```java
package com.mycodeyatra.screenplay.tasks;

import com.mycodeyatra.screenplay.Actor;
import com.mycodeyatra.screenplay.BrowseTheWeb;
import com.mycodeyatra.screenplay.Performable;
import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.support.ui.ExpectedConditions;
import org.openqa.selenium.support.ui.WebDriverWait;

import java.time.Duration;

public class LoginTask implements Performable {
    private final String username;
    private final String password;

    private LoginTask(String username, String password) {
        this.username = username;
        this.password = password;
    }

    public static LoginTask withCredentials(String username, String password) {
        return new LoginTask(username, password);
    }

    @Override
    public void performAs(Actor actor) {
        WebDriver driver = actor.usingAbilityTo(BrowseTheWeb.class).getDriver();
        WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));

        System.out.println("LoginTask: " + actor.getName() + " is entering login credentials...");
        wait.until(ExpectedConditions.visibilityOfElementLocated(By.xpath("//input[@data-testid='username']"))).clear();
        driver.findElement(By.xpath("//input[@data-testid='username']")).sendKeys(username);

        driver.findElement(By.xpath("//input[@data-testid='password']")).clear();
        driver.findElement(By.xpath("//input[@data-testid='password']")).sendKeys(password);

        driver.findElement(By.xpath("//button[@data-testid='login-btn']")).click();
    }
}
```

### 3. Screenplay Question (ProfileHeaderQuestion.java)
Questions encapsulate queries to check element states, text contents, or attributes.

```java
package com.mycodeyatra.screenplay.questions;

import com.mycodeyatra.screenplay.Actor;
import com.mycodeyatra.screenplay.BrowseTheWeb;
import com.mycodeyatra.screenplay.Question;
import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.support.ui.ExpectedConditions;
import org.openqa.selenium.support.ui.WebDriverWait;

import java.time.Duration;

public class ProfileHeaderQuestion implements Question<String> {

    private ProfileHeaderQuestion() {}

    public static ProfileHeaderQuestion value() {
        return new ProfileHeaderQuestion();
    }

    @Override
    public String answeredBy(Actor actor) {
        WebDriver driver = actor.usingAbilityTo(BrowseTheWeb.class).getDriver();
        WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));
        
        System.out.println("ProfileHeaderQuestion: " + actor.getName() + " is retrieving the profile welcome message...");
        return wait.until(ExpectedConditions.visibilityOfElementLocated(By.xpath("//h2[@data-testid='profile-title']"))).getText();
    }
}
```

### 4. Validation Test Suite (ScreenplayTest.java)
Our test cases read like descriptive business specifications:

```java
package com.mycodeyatra.tests;

import com.mycodeyatra.driver.DriverFactory;
import com.mycodeyatra.screenplay.Actor;
import com.mycodeyatra.screenplay.BrowseTheWeb;
import com.mycodeyatra.screenplay.questions.ProfileHeaderQuestion;
import com.mycodeyatra.screenplay.tasks.LoginTask;
import com.mycodeyatra.screenplay.tasks.NavigateTask;
import org.openqa.selenium.WebDriver;
import org.testng.Assert;
import org.testng.annotations.AfterMethod;
import org.testng.annotations.BeforeMethod;
import org.testng.annotations.Optional;
import org.testng.annotations.Parameters;
import org.testng.annotations.Test;

public class ScreenplayTest {

    private WebDriver driver;

    @BeforeMethod
    @Parameters("browser")
    public void setUp(@Optional("chrome") String browser) {
        driver = DriverFactory.initDriver(browser);
        driver.manage().window().maximize();
    }

    @Test
    public void testSuccessfulLoginViaScreenplay() {
        // 1. Instantiating Actor with Abilities
        Actor james = Actor.named("James").can(BrowseTheWeb.with(driver));

        // 2. Perform Tasks
        james.attemptsTo(
                NavigateTask.to("https://practice.mycodeyatra.com/#/login"),
                LoginTask.withCredentials("admin", "admin123")
        );

        // 3. Ask Questions to assert results
        String welcomeMessage = james.asksFor(ProfileHeaderQuestion.value());
        System.out.println("[ScreenplayTest] Welcome Message Retrieved: " + welcomeMessage);
        
        Assert.assertTrue(welcomeMessage.contains("Welcome back, admin!"), 
                "Profile welcome page validation failed in Screenplay run!");
    }

    @AfterMethod
    public void tearDown() {
        DriverFactory.quitDriver();
    }
}
```

---

## 🚀 Benefits of Screenplay Design Pattern

1. **Strict SRP Adherence**: Tasks only carry execution logic. Questions only carry assertion queries. Locators are kept separated. No single class carries multiple responsibilities.
2. **Infinite Reusability**: Unlike Page Objects, Tasks (like LoginTask) are highly modular and can be reused across entirely different features or flows without modifying existing wrappers.
3. **Highly Readable**: Test scripts look like user stories, mapping directly to physical QA tasks.
