---
title: Enterprise Reporting with Serenity BDD and Selenium Java
date: 01-Nov-2024
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [java, selenium, bdd, cucumber, serenity]
category: BDD & Cucumber
categories: [BDD & Cucumber, Java, Automation]
excerpt: >-
  Stop writing boilerplate code! Learn how to use Serenity BDD to automatically manage WebDrivers and generate stunning, step-by-step living documentation reports for your Cucumber test suite.
readTime: 6 min read
---

# Enterprise Reporting with Serenity BDD and Selenium Java

Over the course of this series, we have built a powerful Behavior-Driven Development framework using Gherkin, Cucumber, and PicoContainer. 

However, building a custom framework from scratch takes a lot of time. You have to manually write hooks to manage the WebDriver lifecycle, manually configure Allure or ExtentReports for HTML reporting, and manually write logic to take screenshots when tests fail.

What if there was a library that did **all of this for you automatically**? 

Welcome to **Serenity BDD**.

---

## 1. What is Serenity BDD?

Serenity BDD is an open-source library that acts as a wrapper around Selenium WebDriver and Cucumber. Its primary goal is to generate world-class, living documentation directly from your tests.

### Key Benefits of Serenity:
1. **Zero WebDriver Boilerplate:** You never have to write `driver = new ChromeDriver()` or `driver.quit()`. Serenity manages the browser automatically!
2. **Step-by-Step Screenshots:** Serenity automatically takes a screenshot for *every single step* in your Cucumber scenario, creating a beautiful visual timeline of the test execution.
3. **Living Documentation:** Serenity's HTML reports are legendary. They aggregate your Gherkin feature files, test results, screenshots, and business requirements into a stunning, searchable website that you can host for your Product Managers.

---

## 2. Setting Up Serenity BDD

Setting up Serenity requires a few specific dependencies and plugins in your `pom.xml`.

**Dependencies:**

```xml
<dependencies>
    <dependency>
        <groupId>net.serenity-bdd</groupId>
        <artifactId>serenity-core</artifactId>
        <version>4.0.12</version>
    </dependency>
    <dependency>
        <groupId>net.serenity-bdd</groupId>
        <artifactId>serenity-cucumber</artifactId>
        <version>4.0.12</version>
    </dependency>
</dependencies>
```

**The Serenity Maven Plugin:**
This plugin is required to aggregate the JSON test results into the final beautiful HTML report.

```xml
<build>
    <plugins>
        <plugin>
            <groupId>net.serenity-bdd.maven.plugins</groupId>
            <artifactId>serenity-maven-plugin</artifactId>
            <version>4.0.12</version>
            <executions>
                <execution>
                    <id>serenity-reports</id>
                    <phase>post-integration-test</phase>
                    <goals>
                        <goal>aggregate</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

---

## 3. Writing Serenity Step Definitions

With Serenity, you do not need PicoContainer or Global Hooks to manage the WebDriver! You simply declare a `WebDriver` variable and annotate it with `@Managed`. Serenity will automatically launch the browser before the scenario and close it afterward!

**src/test/java/steps/LoginSteps.java**

```java
package steps;
import io.cucumber.java.en.Given;
import io.cucumber.java.en.When;
import io.cucumber.java.en.Then;
import net.thucydides.core.annotations.Managed;
import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;
import org.testng.Assert;
public class LoginSteps {
    // Serenity automatically initializes and tears down this WebDriver!
    @Managed
    WebDriver driver;
    @Given("the user is on the login page")
    public void navigateToLogin() {
        driver.get("https://mycodeyatra.com/login");
    }
    @When("the user enters the username {string}")
    public void enterUsername(String username) {
        driver.findElement(By.id("username")).sendKeys(username);
    }
    @When("the user enters the password {string}")
    public void enterPassword(String password) {
        driver.findElement(By.id("password")).sendKeys(password);
    }
    @When("the user clicks the login button")
    public void clickLogin() {
        driver.findElement(By.id("login-btn")).click();
    }
    @Then("the user should be redirected to the dashboard")
    public void verifyDashboard() {
        Assert.assertTrue(driver.getCurrentUrl().contains("dashboard"));
    }
}
```

---

## 4. The `serenity.conf` File

To configure Serenity, we use a `serenity.conf` file instead of passing massive command-line arguments. This file allows us to configure environments, WebDriver settings, and headless modes.

**src/test/resources/serenity.conf**

```conf
webdriver {
  driver = chrome
  autodownload = true
  capabilities {
    browserName = "chrome"
    acceptInsecureCerts = true
    "goog:chromeOptions" {
      args = ["remote-allow-origins=*", "test-type", "no-sandbox", "ignore-certificate-errors", "--window-size=1920,1080",
        "incognito", "disable-infobars", "disable-gpu", "disable-default-apps", "disable-popup-blocking"]
    }
  }
}
environments {
  default {
    webdriver.base.url = "https://mycodeyatra.com"
  }
  qa {
    webdriver.base.url = "https://qa.mycodeyatra.com"
  }
  staging {
    webdriver.base.url = "https://staging.mycodeyatra.com"
  }
}
```

---

## 5. Generating the Serenity Report

To run the tests and generate the HTML report, you must run the Maven `verify` lifecycle phase (which triggers the Serenity aggregator plugin we configured earlier).

```bash
mvn clean verify -Denvironment=qa
```

When the execution completes, Serenity will generate a massive static website located at:
`target/site/serenity/index.html`

### What's inside the report?
- **Feature Coverage:** A breakdown of how many scenarios passed, failed, or were ignored.
- **Requirements Traceability:** If you map your feature files to Jira issues (e.g., `@issue:JIRA-123`), Serenity creates direct links to your Jira tickets!
- **Visual Evidence:** If you click on a specific scenario, you will see the exact text of your Gherkin steps, and next to each step, a **screenshot** showing the UI at that exact moment in time!

## Conclusion

Serenity BDD is the gold standard for Enterprise Behavior-Driven Development in Java. By abstracting away WebDriver management and automatically generating living documentation with step-by-step screenshots, it allows QA Engineers to focus entirely on writing business logic rather than writing boilerplate code.

### 🎉 The Java BDD & Cucumber Mastery Series is Complete!
Congratulations! You have now mastered Gherkin syntax, Cucumber Hooks, Tags, PicoContainer Dependency Injection, and Serenity BDD. You are now equipped to build massive, scalable, business-readable automation frameworks!
