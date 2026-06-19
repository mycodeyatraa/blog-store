---
title: The Silent Killer: Automated Flaky Test Detection in TestNG
date: 27-Aug-2026
lastUpdated: 27-Aug-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [java, selenium, reporting, flaky-tests, testng, retry-analyzer, test-automation, debugging]
category: Selenium Java
categories: [Selenium Java, Reporting & Analytics]
excerpt: >-
  Flaky tests destroy developer trust. Learn how to implement a TestNG IRetryAnalyzer and IAnnotationTransformer to automatically retry sporadic failures and surface flaky test debt directly in your Allure dashboard.
readTime: 6 min read
---

# The Silent Killer: Automated Flaky Test Detection in TestNG

In our last tutorial on Dashboard Metrics, we identified that the **Flakiness Rate** is the single most important KPI for an enterprise automation framework. 

A flaky test is a test that randomly fails on the first attempt, but magically passes when executed a second time without any code changes. Flaky tests destroy developer trust. If your CI/CD pipeline fails 20% of the time because of a flaky test, developers will start ignoring the dashboard entirely. 

The easiest way to deal with flakiness is to automatically retry failed tests. In this tutorial, we will learn how to build an automated **RetryAnalyzer** in TestNG to catch flaky tests in the act, and more importantly, how to surface that data in our reports so we can quarantine them!

---

## 1. Implementing the IRetryAnalyzer

TestNG provides a built-in interface called `IRetryAnalyzer`. It allows us to define logic that executes the exact millisecond an `@Test` method fails. If the logic returns `true`, TestNG will ignore the failure and execute the exact same test method again.

Let's create a custom `RetryAnalyzer` class that attempts to rerun a failed test up to a maximum of 3 times.

```java
import org.testng.IRetryAnalyzer;
import org.testng.ITestResult;
public class RetryAnalyzer implements IRetryAnalyzer {
    private int retryCount = 0;
    // Set the maximum number of times to retry a failed test
    private static final int MAX_RETRY_COUNT = 3;
    @Override
    public boolean retry(ITestResult result) {
        if (retryCount < MAX_RETRY_COUNT) {
            retryCount++;
            System.out.println("Retrying test: '" + result.getName() + "' for the " + retryCount + " time.");
            return true; // Return true to tell TestNG to retry the test!
        }
        return false; // Return false to finally accept the failure
    }
}
```

---

## 2. Applying the Retry Logic

Now that we have the logic, we need to apply it to our tests.

**Option A: The Bad Way (Method Level)**
You can apply the retry logic to individual tests using the `retryAnalyzer` attribute in the `@Test` annotation.

```java
@Test(retryAnalyzer = RetryAnalyzer.class)
public void testFlakyLogin() {
    // ...
}
```
*Why is this bad?* Because if you have 5,000 tests, you have to add this parameter to all 5,000 methods. That violates DRY principles.

**Option B: The Enterprise Way (Annotation Transformer)**
To apply the retry analyzer globally to every single test in your entire framework, we must implement a second TestNG listener called `IAnnotationTransformer`. 

This interface allows us to programmatically modify the `@Test` annotations at runtime before the suite executes!

```java
import org.testng.IAnnotationTransformer;
import org.testng.annotations.ITestAnnotation;
import java.lang.reflect.Constructor;
import java.lang.reflect.Method;
public class RetryAnnotationTransformer implements IAnnotationTransformer {
    @Override
    public void transform(ITestAnnotation annotation, Class testClass, Constructor testConstructor, Method testMethod) {
        // If the @Test doesn't already have a retry analyzer defined...
        if (annotation.getRetryAnalyzerClass() == null) {
            // Force it to use our custom RetryAnalyzer!
            annotation.setRetryAnalyzer(RetryAnalyzer.class);
        }
    }
}
```

Now, simply register this transformer in your `testng.xml` file alongside your other listeners:

```xml
<listeners>
    <!-- Register our Failure Screenshot Listener -->
    <listener class-name="com.mycodeyatra.listeners.TestFailureListener" />
    <!-- Register our Global Retry Transformer -->
    <listener class-name="com.mycodeyatra.listeners.RetryAnnotationTransformer" />
</listeners>
```

---

## 3. The Flakiness Data Trap

Congratulations! You have implemented an automated retry loop. If a test fails due to a random network timeout, it will retry and pass. Your CI/CD pipeline is green!

**But there is a massive hidden danger.**

If a test fails on Attempt 1, fails on Attempt 2, but passes on Attempt 3, the default TestNG HTML report will simply mark the test as "Passed".

You have successfully hidden the flaky test from the developers, but you have also hidden it from yourself. If you do not know a test is flaky, you cannot fix it. Eventually, it will start failing on Attempt 3 as well.

---

## 4. Tracking Flakiness in Reporting

To solve this, we must ensure our reporting dashboards explicitly highlight tests that required a retry to pass.

If you are using **Allure Reports**, it handles this perfectly out of the box! 

When you run `allure serve`, Allure looks at the TestNG results. If it sees that a test was executed multiple times with a mix of passes and fails, it assigns a special **"Flaky"** tag (usually a bomb icon 💣) to the test in the UI. 

You can literally filter the Allure dashboard by clicking the "Flaky" checkbox to instantly generate a list of all your technical debt!

### Fixing the Root Cause
When you identify a flaky test on the dashboard, do not ignore it. Investigate it immediately:
1. **Network Issues:** Does the page take a long time to load? Increase your `PageLoadTimeout`.
2. **Dynamic UI Rendering:** Did a button randomly throw `StaleElementReferenceException`? You might need a custom fluent wait to wait for the JavaScript/React DOM to settle.
3. **Database Contention:** Are two tests trying to use the same user account at the same time? Implement independent test data setup via JDBC.

## Conclusion

Automated retry loops are the ultimate double-edged sword. Used correctly, they save your CI/CD pipeline from collapsing due to random network blips. Used incorrectly, they sweep massive synchronization bugs under the rug.

By combining the `IRetryAnalyzer` and `IAnnotationTransformer` with the native Flaky detection built into Allure, you achieve the perfect balance: a green CI/CD pipeline for the developers, and a clear "Hit List" of technical debt for the QA team to fix.

Speaking of technical debt, how do we track our overall automation progress over the course of a year? In our next tutorial, we will explore **Test Analytics**, showing you how to extract historical trend data to prove your framework's long-term value!
