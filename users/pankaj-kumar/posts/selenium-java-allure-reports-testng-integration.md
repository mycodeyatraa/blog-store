---
title: Moving Beyond the Console: Integrating Allure Reports into Selenium
date: 12-Aug-2026
lastUpdated: 12-Aug-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [java, selenium, allure, testng, reporting, test-automation, analytics]
category: Selenium Java
categories: [Selenium Java, Reporting & Analytics]
excerpt: >-
  Stop reading console logs. Learn how to integrate the Allure Framework into your TestNG project to generate beautiful, executive-ready HTML dashboards that categorize your End-to-End test executions.
readTime: 6 min read
---

# Moving Beyond the Console: Integrating Allure Reports into Selenium

Congratulations on reaching Phase 12! Over the past few modules, we have built an incredibly powerful End-to-End automation framework. We are testing the UI, connecting to MySQL, validating Kafka events, and parsing PDFs.

But right now, when our massive test suite finishes, the only output we get is a massive block of green and red text in our IDE console.

If you are a solo developer, the console might be enough. But in an enterprise environment, Project Managers, Product Owners, and Business Analysts do not know how to read an IntelliJ console or a raw TestNG XML file. 

To bridge the gap between engineering and business, we need **Automated Reporting**. In this tutorial, we will learn how to integrate the industry standard **Allure Framework** into our Selenium Java tests to generate beautiful, interactive HTML reports!

---

## 1. What is Allure?

Allure is a lightweight, multi-language test report tool created by Qameta Software. It is designed to show a very concise representation of what has been tested in a neat web report form. 

Unlike the default TestNG reports (which look like they were built in 1995), Allure provides interactive pie charts, timelines, execution trends, and allows you to attach screenshots and logs directly to failing test cases.

---

## 2. Adding the Allure Dependencies

To integrate Allure into a Maven + TestNG project, we need to add the `allure-testng` dependency to our `pom.xml`.

```xml
<dependency>
    <groupId>io.qameta.allure</groupId>
    <artifactId>allure-testng</artifactId>
    <version>2.24.0</version>
    <scope>test</scope>
</dependency>
```

We also need to configure the `maven-surefire-plugin` (which Maven uses to run our TestNG tests) to actually generate the Allure raw results during execution.

Add this to your `pom.xml` under `<build><plugins>`:

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-surefire-plugin</artifactId>
    <version>3.1.2</version>
    <configuration>
        <argLine>
            -javaagent:"${settings.localRepository}/org/aspectj/aspectjweaver/1.9.20.1/aspectjweaver-1.9.20.1.jar"
        </argLine>
    </configuration>
    <dependencies>
        <dependency>
            <groupId>org.aspectj</groupId>
            <artifactId>aspectjweaver</artifactId>
            <version>1.9.20.1</version>
        </dependency>
    </dependencies>
</plugin>
```

---

## 3. Writing an Allure-Annotated Test

Allure works out of the box without any code changes. However, its true power comes from its custom Annotations. 

Let's write a simple Selenium login test and decorate it with Allure annotations to categorize it in our report!

```java
import io.qameta.allure.*;
import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.testng.Assert;
import org.testng.annotations.AfterMethod;
import org.testng.annotations.BeforeMethod;
import org.testng.annotations.Test;
// 1. Class-level Annotations
@Epic("Authentication Module")
@Feature("User Login")
public class AllureBasicsTest {
    WebDriver driver;
    @BeforeMethod
    public void setup() {
        driver = new ChromeDriver();
    }
    // 2. Method-level Annotations
    @Test
    @Story("Successful Login with Valid Credentials")
    @Description("Verifies that a user can login using a valid username and password.")
    @Severity(SeverityLevel.CRITICAL)
    public void testValidLogin() {
        driver.get("https://practice.mycodeyatra.com/login");
        // Let's use the @Step annotation via helper methods
        enterCredentials("testuser", "password123");
        clickLoginButton();
        Assert.assertTrue(driver.findElement(By.id("dashboard")).isDisplayed(),
            "Dashboard did not load after login!");
    }
    // 3. Step-level Annotations
    @Step("Entering username: {0} and password: {1}")
    public void enterCredentials(String username, String password) {
        driver.findElement(By.id("user")).sendKeys(username);
        driver.findElement(By.id("pass")).sendKeys(password);
    }
    @Step("Clicking the Submit Button")
    public void clickLoginButton() {
        driver.findElement(By.id("submit")).click();
    }
    @AfterMethod
    public void teardown() {
        driver.quit();
    }
}
```

---

## 4. Generating the Report

When you run this test (via `mvn clean test`), Allure will silently intercept the TestNG execution and generate a folder named `allure-results` containing a bunch of messy JSON files.

These JSON files are the *raw data*, not the HTML report. 

To view the beautiful HTML report, you must install the **Allure Commandline Tool** on your machine. Once installed, simply open your terminal in the project root and run:

```bash
allure serve allure-results
```

Allure will instantly compile the JSON data into a beautiful HTML website, launch a local web server, and open the report in your default browser! 

You will see pie charts representing pass/fail ratios, a categorized list of tests grouped by `@Epic` and `@Feature`, and a detailed breakdown of every single `@Step` executed during the `testValidLogin` method!

## Conclusion

By integrating Allure into our Selenium framework, we have transformed our automation output from unreadable console logs into gorgeous, executive-ready dashboards.

But what happens when a test fails? Seeing a red "Failed" box on a dashboard is nice, but it doesn't help the engineer debug the issue.

In our next tutorial, we will dive into **Advanced Allure**. We will learn how to configure TestNG Listeners to automatically capture Selenium Screenshots and DOM Page Sources the exact millisecond a test fails, and attach them directly to the Allure Report!
