---
title: Behavior-Driven Development (BDD): Gherkin and Cucumber Basics in Java
date: 29-Oct-2024
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [java, selenium, bdd, cucumber, gherkin]
category: BDD & Cucumber
categories: [BDD & Cucumber, Java, Automation]
excerpt: >-
  Bridge the gap between Business and QA! Learn how to write executable specifications using plain-English Gherkin syntax, and translate them into Selenium Java code using the Cucumber framework.
readTime: 7 min read
---

# Behavior-Driven Development (BDD): Gherkin and Cucumber Basics in Java

For decades, Software Testing suffered from a massive communication gap. 
- Business Analysts wrote requirements in Word documents.
- Developers wrote code in Java.
- QA Engineers wrote test automation in Selenium.

Because everyone spoke a different language, bugs were common, and test automation often tested the *wrong* things. **Behavior-Driven Development (BDD)** was invented to bridge this gap.

In this first article of our Java BDD series, we will teach you how to write executable specifications using **Gherkin**, and how to translate that plain English into Java automation using **Cucumber**.

---

## 1. Introduction to BDD

BDD is an agile software development process that encourages collaboration across all roles (the "Three Amigos": Business, Development, and Testing). 

Instead of writing vague requirements, the team agrees on concrete examples of how the software should behave. These examples are written in a strictly formatted plain-text language called **Gherkin**. 

Because Gherkin is highly structured, automation frameworks (like Cucumber) can read the plain English and execute underlying Java Selenium code!

---

## 2. The Gherkin Syntax and Feature Files

Gherkin files are saved with the `.feature` extension. They use special keywords like `Feature`, `Scenario`, `Given`, `When`, `Then`, and `And`.

Let's look at a standard Feature file for a Login process:

**src/test/resources/features/Login.feature**

```gherkin
Feature: User Authentication
  As a registered user
  I want to log into the application
  So that I can access my dashboard
  Scenario: Successful Login with valid credentials
    Given the user is on the login page
    When the user enters the username "admin"
    And the user enters the password "SuperSecret123!"
    And the user clicks the login button
    Then the user should be redirected to the dashboard
    And the welcome message should be displayed
```

### Why is this powerful?
1. **The Product Manager** can read this and confirm it's exactly what they want.
2. **The Developer** knows exactly what UI elements and backend logic to build.
3. **The QA Engineer** uses this exact file to drive their automated Selenium tests!

---

## 3. Setting Up Cucumber in Java

To make Cucumber understand our `.feature` files, we need to add the necessary dependencies to our `pom.xml`.

```xml
<!-- Cucumber Core and Java -->
<dependency>
    <groupId>io.cucumber</groupId>
    <artifactId>cucumber-java</artifactId>
    <version>7.14.0</version>
    <scope>test</scope>
</dependency>
<!-- Cucumber TestNG Runner -->
<dependency>
    <groupId>io.cucumber</groupId>
    <artifactId>cucumber-testng</artifactId>
    <version>7.14.0</version>
    <scope>test</scope>
</dependency>
<!-- Selenium WebDriver -->
<dependency>
    <groupId>org.seleniumhq.selenium</groupId>
    <artifactId>selenium-java</artifactId>
    <version>4.16.0</version>
</dependency>
```

---

## 4. Writing Step Definitions

Cucumber reads the `.feature` file line by line. For every `Given`, `When`, or `Then` step, Cucumber looks for a matching Java method annotated with a regular expression (or Cucumber Expression) that matches the text.

These Java methods are called **Step Definitions**. This is where we write our Selenium code!

**src/test/java/steps/LoginSteps.java**

```java
package steps;
import io.cucumber.java.en.Given;
import io.cucumber.java.en.When;
import io.cucumber.java.en.Then;
import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.testng.Assert;
public class LoginSteps {
    WebDriver driver;
    @Given("the user is on the login page")
    public void navigateToLoginPage() {
        driver = new ChromeDriver();
        driver.get("https://mycodeyatra.com/login");
    }
    @When("the user enters the username {string}")
    public void enterUsername(String username) {
        // Cucumber automatically extracts "admin" from the Feature file and passes it here!
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
    public void verifyDashboardUrl() {
        Assert.assertTrue(driver.getCurrentUrl().contains("dashboard"));
    }
    @Then("the welcome message should be displayed")
    public void verifyWelcomeMessage() {
        boolean isDisplayed = driver.findElement(By.id("welcome-msg")).isDisplayed();
        Assert.assertTrue(isDisplayed);
        driver.quit();
    }
}
```

### Magic String Extraction
Notice the `{string}` in the `@When` annotation? 
Cucumber's expression engine automatically finds the text `"admin"` in the feature file and passes it as an argument to the Java method! This means you can reuse the same Java method for any username.

---

## 5. The Cucumber TestNG Runner

We have our Feature file, and we have our Step Definitions. Now we need a Runner class to bind them together and execute the test!

**src/test/java/runners/TestRunner.java**

```java
package runners;
import io.cucumber.testng.AbstractTestNGCucumberTests;
import io.cucumber.testng.CucumberOptions;
@CucumberOptions(
    features = "src/test/resources/features",
    glue = "steps",
    plugin = {"pretty", "html:target/cucumber-reports.html"},
    monochrome = true
)
public class TestRunner extends AbstractTestNGCucumberTests {
    // This class is completely empty! 
    // The @CucumberOptions annotation does all the work.
}
```

When you right-click and run `TestRunner.java`, Cucumber will:
1. Scan the `features` folder.
2. Find `Login.feature`.
3. Look inside the `steps` package to find the matching Java methods.
4. Execute the Selenium code line by line.
5. Generate a beautiful HTML report in the `target` folder!

## Conclusion

BDD changes test automation from a purely technical exercise into a collaborative business process.
- **Gherkin** acts as the living documentation that everyone can read.
- **Cucumber** acts as the engine that parses the Gherkin.
- **Step Definitions** act as the glue that binds plain English to Selenium Java code.

In our next article, we will tackle advanced Cucumber architecture: **Hooks**, **Tags**, and how to cleanly share state between different Step Definition classes using **Dependency Injection (PicoContainer)**!
