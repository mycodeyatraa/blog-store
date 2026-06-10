---
title: Assertions in Test Automation: Hard, Soft, and Fluent Assertions
date: 07-Jul-2024
lastUpdated: 10-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [selenium, java, testng, assertj]
category: UI Automation
categories: [UI Automation, Selenium, Java]
excerpt: >-
  An automated test without assertions is not a test. Learn the differences between Hard, Soft, and Fluent AssertJ assertions, when to use each style, and common design pitfalls.
readTime: 6 min read
---

# Assertions in Test Automation: Hard, Soft, and Fluent Assertions

> 📅 **Last Updated:** 10-Jun-2026

An automated test without assertions is not actually a test—it is merely a script that runs through actions. Assertions define the success criteria of your tests, verifying that the actual application state matches your expected business outcomes.

In this guide, we will explore the three main assertion strategies used in modern Java test frameworks: **Hard Assertions**, **Soft Assertions**, and **Fluent Assertions (AssertJ)**. We will compare their behavior, look at code examples, and highlight how to choose the right strategy.

---

## 🚦 Execution Flow: Hard vs. Soft Assertions

The core difference between Hard and Soft assertions lies in how they handle execution failures during a test:

![diagram_1](https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/posts/assertions-testng-junit-assertj-guide/images/diagram_1.png)

---

## 🛠️ The Three Assertion Strategies

### 1. Hard Assertions (The Default Choice)
Provided by test runners like TestNG (`org.testng.Assert`) and JUnit (`org.junit.jupiter.api.Assertions`). If a hard assertion fails, it throws an `AssertionError` immediately, halting the execution of that specific test method. Remaining steps in the test block are skipped.

* **Use Case**: Critical verification steps (e.g., verifying login success before testing user dashboard settings).

```java
import org.testng.Assert;
import org.testng.annotations.Test;
public class HardAssertTest {
    @Test
    public void testLoginFlow() {
        String pageTitle = "Dashboard";
        Assert.assertEquals(pageTitle, "Dashboard", "Page title does not match!");
        System.out.println("This line executes only if the assertion passes.");
    }
}
```

### 2. Soft Assertions (Accumulated Verification)
Provided by TestNG (`org.testng.asserts.SoftAssert`). Soft assertions collect failures during the test run but allow execution to continue. This is highly useful for validating multiple unrelated checkpoints in a single workflow.

> [!IMPORTANT]
> You **must** call `.assertAll()` at the end of the test. If you omit this call, the test will pass successfully even if several soft assertions failed!

```java
import org.testng.annotations.Test;
import org.testng.asserts.SoftAssert;
public class SoftAssertTest {
    @Test
    public void testUserProfileDashboard() {
        SoftAssert softAssert = new SoftAssert();
        String username = "JohnDoe";
        String role = "Manager";
        softAssert.assertEquals(username, "JohnDoe", "Username mismatch!");
        softAssert.assertEquals(role, "Admin", "User role mismatch!"); // Fails, but test continues
        System.out.println("This step runs despite the role mismatch.");
        softAssert.assertAll(); // Flunks the test here and prints all accumulated errors
    }
}
```

### 3. Fluent Assertions (AssertJ)
AssertJ is a popular Java library that provides fluent, natural-language assertions. It drastically increases code readability and produces clean, descriptive failure messages out-of-the-box.

* **Readability**: Code reads like a natural English sentence (`assertThat(title).isEqualTo("Dashboard")`).
* **Method Chaining**: Chain multiple checks on the same object (e.g., verifying a string is not null, starts with a prefix, and contains specific substrings).

```java
import static org.assertj.core.api.Assertions.assertThat;
import org.testng.annotations.Test;
import java.util.List;
public class AssertJTest {
    @Test
    public void testDashboardData() {
        String welcomeMessage = "Welcome, Pankaj Kumar!";
        assertThat(welcomeMessage)
            .isNotNull()
            .startsWith("Welcome")
            .contains("Pankaj");
        List<String> activeModules = List.of("Settings", "Reports", "Analytics");
        assertThat(activeModules)
            .hasSize(3)
            .contains("Reports")
            .doesNotContain("Admin Panel");
    }
}
```
### 4. Test Suite Execution Logs

When running this suite through Maven (`mvn test -Dtest=AssertionsDemoTest`), we see that `testAssertJAssertion` and `testHardAssertion` pass successfully, while `testSoftAssertion` fails and dumps the accumulated error report at the end because of `.assertAll()`:

```text
-------------------------------------------------------
 T E S T S
-------------------------------------------------------
Running com.mycodeyatra.tests.AssertionsDemoTest
[Assertions] Executing AssertJ Fluent Assertion Test...
[Assertions] AssertJ fluent assertions passed successfully.
[Assertions] Executing Hard Assertion Test...
[Assertions] Hard assertion passed successfully.
[Assertions] Executing Soft Assertion Test...
[Assertions] First soft assertion checked.
[Assertions] Second soft assertion checked (intended failure).
[Assertions] Finalizing soft assertions via assertAll...
Tests run: 3, Failures: 1, Errors: 0, Skipped: 0, Time elapsed: 6.82 sec <<< FAILURE!
testSoftAssertion(com.mycodeyatra.tests.AssertionsDemoTest)  Time elapsed: 0.334 sec  <<< FAILURE!
java.lang.AssertionError: The following asserts failed:
	User role mismatch! expected [Admin] but found [Manager]
	at org.testng.asserts.SoftAssert.assertAll(SoftAssert.java:46)
	at org.testng.asserts.SoftAssert.assertAll(SoftAssert.java:30)
	at com.mycodeyatra.tests.AssertionsDemoTest.testSoftAssertion(AssertionsDemoTest.java:34)
Results :
Failed tests:   testSoftAssertion(com.mycodeyatra.tests.AssertionsDemoTest): The following asserts failed:(..)
Tests run: 3, Failures: 1, Errors: 0, Skipped: 0
```

---


## 📊 Comparison of Assertion Types

| Feature | Hard Assertions (TestNG/JUnit) | Soft Assertions (TestNG) | Fluent Assertions (AssertJ) |
| :--- | :--- | :--- | :--- |
| **Failure Behavior** | Halts test execution instantly. | Collects failures, continues running. | Halts test execution instantly. |
| **Readability** | Standard prefix style (`assertEquals(act, exp)`). | Standard prefix style. | Fluent readable syntax (`assertThat(act).isEqualTo(exp)`). |
| **Compilation** | Simple. | Requires instantiating a `SoftAssert` helper object. | Requires importing AssertJ library dependencies. |
| **Common Pitfall** | Unreachable code after assertion failure. | Forgetting to call `.assertAll()` at the end. | Importing incorrect static classes. |
| **Best Used For** | Critical gatekeeping verification. | Forms validation, dashboard UI layouts. | Production-grade framework assertions. |

---

## 💡 Best Practices for Assertions

1. **Write Descriptive Messages**: Always use the optional `message` parameter in assertions. If a test fails in headless mode on a CI server, clear custom messages save hours of debugging time.
2. **One Logical Outcome Per Test**: Avoid writing long tests that verify 20 different behaviors. Break them down, or use soft assertions only when validating a single entity's multi-property state (like a profile form submission).
3. **Prefer AssertJ for Complex Objects**: When validating lists, collections, maps, or custom data structures, AssertJ's fluent matchers provide much stronger verification capabilities than standard TestNG assertions.
