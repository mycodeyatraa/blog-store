---
title: The Original Cloud Grid: Executing Tests with Sauce Labs
date: 05-Oct-2026
lastUpdated: 05-Oct-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [java, selenium, sauce-labs, cloud, remote-webdriver, grid, managed-infrastructure, debugging]
category: Selenium Java
categories: [Selenium Java, Cloud & CI/CD]
excerpt: >-
  Switch cloud providers seamlessly. Learn how to route your W3C RemoteWebDriver commands to Sauce Labs, utilizing their extended debugging features to automatically capture Network HAR files and video recordings.
readTime: 6 min read
---

# The Original Cloud Grid: Executing Tests with Sauce Labs

In our previous tutorial, we integrated our TestNG framework with BrowserStack. BrowserStack is incredibly popular, but it wasn't the first. 

The original pioneer of the Cloud Selenium Grid is **Sauce Labs**. In fact, Sauce Labs was co-founded by Jason Huggins, the original creator of Selenium itself! 

In an enterprise environment, your company might have an exclusive contract with Sauce Labs instead of BrowserStack. As a Senior SDET, you must be able to swap out cloud providers with minimal code changes. 

In this tutorial, we will learn how to point our `RemoteWebDriver` to Sauce Labs and utilize their advanced video debugging tools.

---

## 1. Setting Up Sauce Labs Credentials

Just like BrowserStack, Sauce Labs requires authentication. 

Create an account, log into the Sauce Labs dashboard, and navigate to "User Settings". You will find your **Username** and **Access Key**. 

Store these securely in your Environment Variables or CI/CD pipeline secrets:
*   `SAUCE_USERNAME`
*   `SAUCE_ACCESS_KEY`

---

## 2. Configuring RemoteWebDriver for Sauce Labs

The beauty of the W3C WebDriver standard is that the core Java code remains exactly the same. We only need to change the Grid URL and the specific cloud platform options (`sauce:options`).

Update your `BaseTest` or `WebDriverManager` class:

```java
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.remote.DesiredCapabilities;
import org.openqa.selenium.remote.RemoteWebDriver;
import java.net.URL;
import java.util.HashMap;
public class BaseTest {
    protected ThreadLocal<WebDriver> driver = new ThreadLocal<>();
    public void setupSauceLabs() throws Exception {
        String username = System.getenv("SAUCE_USERNAME");
        String accessKey = System.getenv("SAUCE_ACCESS_KEY");
        // The URL provided by Sauce Labs (EU or US Datacenter)
        String gridUrl = "https://" + username + ":" + accessKey + "@ondemand.us-west-1.saucelabs.com:443/wd/hub";
        DesiredCapabilities capabilities = new DesiredCapabilities();
        // 1. Specify the OS and Browser you want to rent
        capabilities.setCapability("browserName", "firefox");
        capabilities.setCapability("browserVersion", "latest");
        capabilities.setCapability("platformName", "Windows 10");
        // 2. Sauce Labs specific options (sauce:options)
        HashMap<String, Object> sauceOptions = new HashMap<>();
        // Organize your tests in the Sauce Labs UI
        sauceOptions.put("name", "Firefox Regression - Login");
        sauceOptions.put("build", "Release-1.0.45");
        sauceOptions.put("tags", new String[]{"regression", "firefox", "auth"});
        // Advanced Debugging Features
        sauceOptions.put("recordVideo", true);
        sauceOptions.put("recordScreenshots", true);
        sauceOptions.put("extendedDebugging", true); // Captures HAR network files
        capabilities.setCapability("sauce:options", sauceOptions);
        // 3. Connect to the Cloud!
        driver.set(new RemoteWebDriver(new URL(gridUrl), capabilities));
    }
}
```

### What is `extendedDebugging`?
When you set `extendedDebugging` to `true`, Sauce Labs automatically captures a `.har` (HTTP Archive) file of all network traffic that occurs during your test. If your UI test fails because a backend API returned a 500 Error, you can literally download the network logs directly from the Sauce Labs dashboard and attach them to your Jira ticket!

---

## 3. Passing the TestNG Status to Sauce Labs

Just like BrowserStack, Sauce Labs has no idea if your `Assert.assertTrue()` passed or failed. It only knows if the browser closed successfully.

We must use the Javascript Executor API to explicitly pass the TestNG status to the Sauce Labs dashboard.

Add this logic to your custom `ITestListener`:

```java
import org.openqa.selenium.JavascriptExecutor;
import org.openqa.selenium.WebDriver;
import org.testng.ITestListener;
import org.testng.ITestResult;
public class SauceLabsListener implements ITestListener {
    @Override
    public void onTestSuccess(ITestResult result) {
        updateSauceLabsStatus(result, true);
    }
    @Override
    public void onTestFailure(ITestResult result) {
        updateSauceLabsStatus(result, false);
    }
    private void updateSauceLabsStatus(ITestResult result, boolean hasPassed) {
        // Retrieve the thread-safe driver from BaseTest
        WebDriver driver = BaseTest.getDriver(); 
        if (driver instanceof JavascriptExecutor) {
            JavascriptExecutor jse = (JavascriptExecutor) driver;
            // Sauce Labs uses a very simple boolean flag
            String script = "sauce:job-result=" + (hasPassed ? "passed" : "failed");
            jse.executeScript(script);
            // Optionally, we can also pass the exact error message
            if (!hasPassed) {
                String errorMsg = result.getThrowable().getMessage().replace("\"", "'");
                jse.executeScript("sauce:context=Failure Reason: " + errorMsg);
            }
        }
    }
}
```

With this listener in place, the moment a test fails in TestNG, your Java code instantly injects the "failed" status and the specific error message into the Sauce Labs UI.

## Conclusion

BrowserStack and Sauce Labs solve the exact same problem: Abstracting infrastructure away from the QA team. 

The W3C standard guarantees that switching between these multi-million dollar cloud providers is as simple as changing the `gridUrl` string and swapping `bstack:options` for `sauce:options`. 

However, there is a third, newer player in the Cloud Grid market that has aggressively captured market share by focusing entirely on Execution Speed and AI.

In our next tutorial, we will explore the fastest growing platform in the industry: **LambdaTest**!
