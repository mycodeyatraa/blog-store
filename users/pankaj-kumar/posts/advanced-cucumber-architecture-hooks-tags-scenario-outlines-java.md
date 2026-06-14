---
title: Advanced Cucumber Architecture: Hooks, Tags, and Scenario Outlines
date: 30-Oct-2024
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [java, selenium, bdd, cucumber, hooks, tags]
category: BDD & Cucumber
categories: [BDD & Cucumber, Java, Automation]
excerpt: >-
  Upgrade your Cucumber framework! Learn how to data-drive tests with Scenario Outlines, filter executions via Tags, and globally manage WebDriver sessions using @Before and @After Hooks.
readTime: 6 min read
---

# Advanced Cucumber Architecture: Hooks, Tags, and Scenario Outlines

In our previous article, we learned how to write a basic Gherkin Scenario and bind it to Selenium Java code using Step Definitions. But Enterprise frameworks require much more power. 

How do we run the same test with 50 different users? How do we organize our tests so we can run a quick "Smoke Test" before a full "Regression"? How do we automatically take screenshots if a test fails without writing repetitive code?

In this article, we will upgrade your Cucumber framework by mastering **Scenario Outlines**, **Tags**, and **Hooks**!

---

## 1. Data-Driven Testing with Scenario Outlines

If you want to test the Login page with three different sets of credentials, you *could* write three separate Scenarios. But that violates the DRY (Don't Repeat Yourself) principle.

Instead, we use a **Scenario Outline** and an **Examples** table.

**LoginDataDriven.feature**

```gherkin
Feature: Data-Driven Authentication
  Scenario Outline: Validate Login with multiple user roles
    Given the user is on the login page
    When the user enters the username "<Username>"
    And the user enters the password "<Password>"
    And the user clicks the login button
    Then the system should display the message "<ExpectedMessage>"
    Examples:
      | Username | Password | ExpectedMessage     |
      | admin    | pass123  | Dashboard           |
      | customer | pass456  | Welcome to Shop     |
      | hacker   | wrong!   | Invalid credentials |
```

When Cucumber sees this `Scenario Outline`, it will automatically execute the entire block **three times**, substituting the `<Variables>` with the values from the `Examples` table!

### Updating the Step Definition
Your Java code doesn't need to change much. It simply accepts the parameters from the feature file!

```java
@When("the user enters the username {string}")
public void enterUsername(String user) {
    driver.findElement(By.id("username")).sendKeys(user);
}
```

---

## 2. Filtering Execution with Cucumber Tags

As your framework grows, you will eventually have hundreds of feature files. You don't want to run all of them every time a developer commits code.

You can categorize your scenarios using **Tags**.

**Checkout.feature**

```gherkin
@E2E @Regression
Feature: E-Commerce Checkout
  @Smoke
  Scenario: Guest user can successfully purchase an item
    Given ...
  @EdgeCase
  Scenario: User tries to checkout with expired credit card
    Given ...
```

### Running Specific Tags via Maven
If the developers just pushed a hotfix and you only want to run the critical `@Smoke` tests, you can trigger Cucumber via the Maven command line and pass a tags filter!

```bash
mvn test -Dcucumber.filter.tags="@Smoke"
```
You can also use logical operators!
- Run both: `-Dcucumber.filter.tags="@Smoke or @Regression"`
- Exclude a tag: `-Dcucumber.filter.tags="not @EdgeCase"`

---

## 3. Global Setup and Teardown with Hooks

In our basic framework, we initialized the `WebDriver` inside the `@Given` step, and called `driver.quit()` inside the `@Then` step. 

**This is a terrible anti-pattern.** What if the test fails at the `@When` step? The `@Then` step will never execute, the browser will stay open forever, and you will eventually crash your server with thousands of orphaned Chrome processes!

Instead, we should use Cucumber **Hooks** (`@Before` and `@After`) to manage the WebDriver lifecycle globally.

**src/test/java/hooks/GlobalHooks.java**

```java
package hooks;
import io.cucumber.java.After;
import io.cucumber.java.Before;
import io.cucumber.java.Scenario;
import org.openqa.selenium.OutputType;
import org.openqa.selenium.TakesScreenshot;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
public class GlobalHooks {
    // Ideally, this WebDriver should be shared using Dependency Injection (Covered in the next article!)
    public static WebDriver driver;
    @Before
    public void setupBrowser() {
        System.out.println("Initializing WebDriver before scenario...");
        driver = new ChromeDriver();
        driver.manage().window().maximize();
    }
    @After
    public void teardownAndScreenshot(Scenario scenario) {
        if (scenario.isFailed()) {
            System.out.println("Scenario Failed! Capturing Screenshot...");
            final byte[] screenshot = ((TakesScreenshot) driver).getScreenshotAs(OutputType.BYTES);
            // Embed the screenshot directly into the Cucumber HTML report!
            scenario.attach(screenshot, "image/png", scenario.getName()); 
        }
        if (driver != null) {
            driver.quit();
            System.out.println("Browser closed safely.");
        }
    }
}
```

### Tagged Hooks
Sometimes you need special setup logic only for certain tests. For example, maybe you need to initialize a Database Connection only for tests tagged with `@Database`.

```java
@Before("@Database")
public void connectToDb() {
    System.out.println("Connecting to SQL Database...");
}
```

## Conclusion

By incorporating these three advanced Cucumber features, you transform your test scripts into a highly organized, professional framework:
- **Scenario Outlines** let you test massive datasets cleanly.
- **Tags** let you slice and dice your test suite for CI/CD pipelines.
- **Hooks** guarantee that your WebDriver initializes properly and cleans itself up gracefully, automatically attaching beautiful screenshots to your reports when things go wrong!

In our next article, we will solve the hardest problem in Cucumber: **Sharing State**. We will teach you how to pass the WebDriver and Test Data between multiple different Step Definition classes using **Dependency Injection (PicoContainer)**!
