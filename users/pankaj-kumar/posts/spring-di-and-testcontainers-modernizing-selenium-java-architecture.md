---
title: Spring DI & Testcontainers: Modernizing Selenium Java Architecture
date: 05-Aug-2024
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [selenium, java, spring-boot, testcontainers, docker, automation]
category: UI Automation
categories: [UI Automation, Selenium, Java]
excerpt: >-
  Replace rigid static factories with Spring Dependency Injection (DI) and fragile grids with Testcontainers. Learn how to spin up ephemeral Docker environments on the fly for scalable UI automation.
readTime: 5 min read
---

# Spring DI & Testcontainers: Modernizing Selenium Java Architecture

If your test framework relies on static blocks to manage the WebDriver instance and requires a constantly running Selenium Grid to execute tests, you are bound to face flakiness. Parallel execution becomes a nightmare of thread-local leaks, and onboarding new engineers requires complex local setup.

In this article, we replace rigid, static factories with **Spring Dependency Injection (DI)** and replace fragile, permanent grids with **Testcontainers** — spinning up ephemeral, disposable browser environments on the fly.

---

## Why Spring and Testcontainers?

1. **Spring Dependency Injection (DI):** Manages the lifecycle of your Page Objects, Configuration Beans, and WebDriver instances. No more `new LoginPage(driver)` cluttering your tests.
2. **Testcontainers:** Uses Docker APIs to spin up temporary Selenium Grid containers. The browser is created when the test starts and destroyed when it finishes. Zero local setup required.

![diagram_1](https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/posts/spring-di-and-testcontainers-modernizing-selenium-java-architecture/images/diagram_1.png)

---

## 1. Setting Up the Dependencies

Add Spring Boot Test and Testcontainers to your `pom.xml`:

```xml
<!-- Spring Boot for Dependency Injection -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <version>3.2.0</version>
</dependency>
 
<!-- Testcontainers for Selenium -->
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>selenium</artifactId>
    <version>1.19.3</version>
</dependency>
```

---

## 2. Managing the Browser Lifecycle with Testcontainers

Instead of pointing your RemoteWebDriver to a hardcoded `localhost:4444`, we let Testcontainers boot a pristine Chrome container for our test session.

```java
package com.mycodeyatra.config;
 
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeOptions;
import org.openqa.selenium.remote.RemoteWebDriver;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Scope;
import org.testcontainers.containers.BrowserWebDriverContainer;
 
@Configuration
public class WebDriverConfig {
 
    @Bean(initMethod = "start", destroyMethod = "stop")
    public BrowserWebDriverContainer<?> browserContainer() {
        return new BrowserWebDriverContainer<>()
            .withCapabilities(new ChromeOptions());
    }
 
    @Bean
    @Scope("cucumber-glue") // Or "prototype" for TestNG
    public WebDriver webDriver(BrowserWebDriverContainer<?> container) {
        return new RemoteWebDriver(container.getSeleniumAddress(), new ChromeOptions());
    }
}
```

> **Key insight:** The `@Bean` lifecycle hooks (`initMethod="start"`, `destroyMethod="stop"`) mean Spring automatically handles starting the Docker container before the driver is created, and tearing it down after the context closes.

---

## 3. Creating Spring-Managed Page Objects

By annotating your Page Objects with `@Component`, Spring automatically injects the WebDriver instance into them.

```java
package com.mycodeyatra.pages;
 
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.support.FindBy;
import org.openqa.selenium.support.PageFactory;
import org.springframework.stereotype.Component;
 
@Component
public class LoginPage {
 
    private final WebDriver driver;
 
    @FindBy(id = "username")
    private WebElement usernameInput;
 
    @FindBy(id = "password")
    private WebElement passwordInput;
 
    @FindBy(id = "loginBtn")
    private WebElement loginButton;
 
    // Spring injects the WebDriver automatically
    public LoginPage(WebDriver driver) {
        this.driver = driver;
        PageFactory.initElements(driver, this);
    }
 
    public void login(String user, String pass) {
        usernameInput.sendKeys(user);
        passwordInput.sendKeys(pass);
        loginButton.click();
    }
}
```

---

## 4. The Test Class: Orchestration Only

With Spring DI handling the wiring, your test class becomes incredibly clean. You simply `@Autowired` the pages you need.

```java
package com.mycodeyatra.tests;
 
import com.mycodeyatra.pages.LoginPage;
import com.mycodeyatra.pages.DashboardPage;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.testng.Assert;
import org.testng.annotations.Test;
 
@SpringBootTest(classes = com.mycodeyatra.config.TestApplication.class)
public class LoginTest extends SpringBaseTest {
 
    @Autowired
    private LoginPage loginPage;
 
    @Autowired
    private DashboardPage dashboardPage;
 
    @Test
    public void testSuccessfulLoginInContainer() {
        driver.get("https://practice.mycodeyatra.com/login");
 
        loginPage.login("admin", "admin123");
 
        Assert.assertTrue(dashboardPage.isDisplayed(), "Dashboard should be visible.");
    }
}
```

---

## Testcontainers Lifecycle in Action

Here is what happens behind the scenes the moment you execute the test:

![diagram_2](https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/posts/spring-di-and-testcontainers-modernizing-selenium-java-architecture/images/diagram_2.png)

## Key Advantages of this Architecture

1. **Zero State Bleed:** Because the container is destroyed after the execution context closes, every test gets a 100% clean browser profile. No leftover cookies, cache, or local storage.
2. **Guaranteed Consistency:** "It works on my machine" is eliminated. The browser running on the developer's laptop is the exact same Docker image running in the CI/CD pipeline.
3. **No Flaky Grids:** You no longer need to maintain a persistent Selenium Grid that crashes or runs out of memory over time.
4. **No Static Variables:** Spring handles dependency graphs elegantly, making parallel execution completely thread-safe without manual `ThreadLocal` management.

By shifting to Spring DI and Testcontainers, you graduate from writing test scripts to engineering a modern, scalable test infrastructure. In our next article, we'll dive into building **Custom TestNG Listeners** for advanced reporting and retry logic.
