---
title: Sharing State in Cucumber using Dependency Injection (PicoContainer)
date: 31-Oct-2024
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [java, selenium, bdd, cucumber, picocontainer, dependency-injection]
category: BDD & Cucumber
categories: [BDD & Cucumber, Java, Automation]
excerpt: >-
  Solve the hardest problem in BDD! Learn how to safely share WebDrivers and Test Data across multiple Step Definition classes using Dependency Injection with cucumber-picocontainer.
readTime: 6 min read
---

# Sharing State in Cucumber using Dependency Injection (PicoContainer)

When you first start learning Cucumber, you usually write all your Step Definitions inside a single Java class. But as your Enterprise framework grows to 500+ scenarios, putting everything in one massive `Steps.java` file becomes impossible to maintain.

You must split your steps into logical classes: `LoginSteps.java`, `CartSteps.java`, `PaymentSteps.java`. 

But this introduces the hardest architectural problem in BDD: **How do multiple Step Definition classes share the exact same `WebDriver` instance?**

In this article, we will teach you why `public static WebDriver driver` is a terrible anti-pattern, and how to elegantly share state across your Cucumber framework using **Dependency Injection** with **PicoContainer**.

---

## 1. The Problem with `static` Variables

Beginners often try to solve the sharing problem by making the WebDriver `static`.

```java
// Anti-Pattern! Do NOT do this!
public class BaseTest {
    public static WebDriver driver;
}
```

If you use `static`, the WebDriver belongs to the *class*, not the *thread*. As soon as you try to run your Cucumber tests in **parallel** (running 5 tests at the same time to speed up execution), the threads will overwrite each other's `static` variables, causing massive crashes and cross-browser contamination!

---

## 2. The Solution: Dependency Injection (DI)

Dependency Injection is a design pattern where an external container creates an object (like our WebDriver) and "injects" it into any class that asks for it via its constructor.

For Cucumber, the officially supported, lightweight DI container is **PicoContainer**.

### Step 1: Add the PicoContainer Dependency

Add the following to your `pom.xml`:

```xml
<dependency>
    <groupId>io.cucumber</groupId>
    <artifactId>cucumber-picocontainer</artifactId>
    <version>7.14.0</version>
    <scope>test</scope>
</dependency>
```

---

## 3. Creating the Test Context

Instead of sharing just the `WebDriver`, it is best practice to create a wrapper class called `TestContext`. This class will hold the `WebDriver`, Page Objects, and any test data you want to share between steps (like a generated Order ID).

**src/test/java/context/TestContext.java**

```java
package context;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
public class TestContext {
    private WebDriver driver;
    private String sharedOrderId;
    // The WebDriver is initialized ONCE when the Context is created
    public WebDriver getDriver() {
        if (driver == null) {
            driver = new ChromeDriver();
            driver.manage().window().maximize();
        }
        return driver;
    }
    public void setSharedOrderId(String sharedOrderId) {
        this.sharedOrderId = sharedOrderId;
    }
    public String getSharedOrderId() {
        return sharedOrderId;
    }
}
```

---

## 4. Injecting the Context into Step Definitions

Now, look at how clean our Step Definitions become. We do not use `static`, and we do not use `new ChromeDriver()`. We simply declare a constructor that accepts `TestContext`. 

PicoContainer detects this constructor and automatically passes the exact same `TestContext` instance to every Step class used in the current Scenario!

**src/test/java/steps/CartSteps.java**

```java
package steps;
import context.TestContext;
import io.cucumber.java.en.Given;
public class CartSteps {
    private TestContext testContext;
    // PicoContainer automatically injects the TestContext here!
    public CartSteps(TestContext testContext) {
        this.testContext = testContext;
    }
    @Given("the user adds an item to the cart")
    public void addItemToCart() {
        // Use the shared WebDriver!
        testContext.getDriver().get("https://mycodeyatra.com/store");
        // ... selenium code ...
        // Save test data to the shared context!
        testContext.setSharedOrderId("ORD-9999");
    }
}
```

Now, let's look at the `PaymentSteps.java` class. It also asks for the `TestContext`. PicoContainer is smart enough to know it's the *same scenario*, so it hands `PaymentSteps` the exact same `TestContext` object!

**src/test/java/steps/PaymentSteps.java**

```java
package steps;
import context.TestContext;
import io.cucumber.java.en.Then;
import org.testng.Assert;
public class PaymentSteps {
    private TestContext testContext;
    // PicoContainer injects the SAME TestContext instance!
    public PaymentSteps(TestContext testContext) {
        this.testContext = testContext;
    }
    @Then("the order is placed successfully")
    public void verifyOrder() {
        // We retrieve the Order ID that was saved by CartSteps!
        String orderId = testContext.getSharedOrderId();
        System.out.println("Verifying Payment for Order: " + orderId);
        Assert.assertEquals(orderId, "ORD-9999");
        // We use the exact same WebDriver instance!
        Assert.assertTrue(testContext.getDriver().getCurrentUrl().contains("success"));
    }
}
```

---

## 5. Cleaning Up via Hooks

Since `TestContext` manages the WebDriver, we should use a Cucumber `@After` hook to safely close it when the Scenario finishes. The Hook class *also* asks PicoContainer for the `TestContext`!

**src/test/java/hooks/GlobalHooks.java**

```java
package hooks;
import context.TestContext;
import io.cucumber.java.After;
public class GlobalHooks {
    private TestContext testContext;
    public GlobalHooks(TestContext testContext) {
        this.testContext = testContext;
    }
    @After
    public void tearDown() {
        if (testContext.getDriver() != null) {
            testContext.getDriver().quit();
        }
    }
}
```

## Conclusion

By using `cucumber-picocontainer`:
1. You **eliminate the `static` keyword**, making your framework 100% thread-safe for parallel execution.
2. You can freely split your massive Step Definition files into dozens of smaller, highly organized, Domain-Specific classes (`UserSteps.java`, `ApiSteps.java`, `DbSteps.java`).
3. You can cleanly pass data (like Order IDs, generated emails, or Page Objects) across different steps using a `TestContext` wrapper.

In our next and final article for this series, we will explore **Serenity BDD**, a powerful wrapper around Cucumber that automatically generates some of the most beautiful, living-documentation test reports in the industry!
