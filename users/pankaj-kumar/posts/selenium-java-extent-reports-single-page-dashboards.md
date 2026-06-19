---
title: Building Custom Dashboards: Integrating Extent Reports in Selenium
date: 18-Aug-2026
lastUpdated: 18-Aug-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [java, selenium, extent-reports, testng, reporting, thread-safe, test-automation]
category: Selenium Java
categories: [Selenium Java, Reporting & Analytics]
excerpt: >-
  When you need an automated dashboard that you can attach to an email without command-line dependencies, Extent Reports is the answer. Learn how to build a Thread-Safe ExtentManager and custom TestNG Listener.
readTime: 6 min read
---

# Building Custom Dashboards: Integrating Extent Reports in Selenium

In our previous tutorials, we integrated the Allure framework into our automation suite. Allure is fantastic, but it has one major drawback: it requires a dedicated command-line server (`allure serve`) or a CI/CD plugin to generate and view the HTML report from the raw JSON files. 

What if you want a beautiful, single-page HTML report that you can instantly attach to an email or send to a Slack channel the second the test suite finishes?

Enter **Extent Reports**.

Extent Reports is a purely programmatic reporting library. It doesn't rely on hidden plugins or secondary command-line tools. You control the exact creation of the HTML file using Java code!

---

## 1. Adding Extent Reports to Maven

Let's add the official `extentreports` dependency to our `pom.xml`.

```xml
<dependency>
    <groupId>com.aventstack</groupId>
    <artifactId>extentreports</artifactId>
    <version>5.1.1</version>
</dependency>
```

*(Note: There are no extra Maven plugins required! Just this single dependency).*

---

## 2. The Core Architecture of Extent Reports

Unlike Allure, which relies on `@Annotations`, Extent Reports requires you to explicitly write Java code to start the report, create tests, and log steps. 

It revolves around three core classes:
1. **`ExtentSparkReporter`**: Defines the physical HTML file that will be generated and configures its theme (Dark/Light).
2. **`ExtentReports`**: The main engine. You attach the SparkReporter to this engine.
3. **`ExtentTest`**: Represents a single test case within the report. You use this object to log steps (Pass/Fail/Info) and attach screenshots.

---

## 3. Creating a Thread-Safe Report Manager

If you are running your Selenium tests in parallel (e.g., 5 browsers at once), you must ensure that your Extent Reports implementation is Thread-Safe, or else the test logs will overwrite each other!

Let's create a singleton `ExtentManager` to initialize the HTML file:

```java
import com.aventstack.extentreports.ExtentReports;
import com.aventstack.extentreports.reporter.ExtentSparkReporter;
import com.aventstack.extentreports.reporter.configuration.Theme;
public class ExtentManager {
    private static ExtentReports extent;
    public static synchronized ExtentReports getInstance() {
        if (extent == null) {
            // 1. Define the HTML file location
            String reportPath = System.getProperty("user.dir") + "/target/ExtentReport.html";
            ExtentSparkReporter spark = new ExtentSparkReporter(reportPath);
            // 2. Configure the UI Theme
            spark.config().setTheme(Theme.DARK);
            spark.config().setDocumentTitle("Enterprise Automation Dashboard");
            spark.config().setReportName("Regression Suite Results");
            // 3. Attach the Reporter to the Engine
            extent = new ExtentReports();
            extent.attachReporter(spark);
            // 4. Add System Variables to the Dashboard
            extent.setSystemInfo("OS", System.getProperty("os.name"));
            extent.setSystemInfo("QA Engineer", "MyCodeYatra Automation Bot");
            extent.setSystemInfo("Environment", "QA Staging");
        }
        return extent;
    }
}
```

---

## 4. Building the Extent Listener

Just like we did with Allure, we will use a **TestNG `ITestListener`** to automatically hook our Extent Reports logic into the test execution lifecycle.

We will use a `ThreadLocal<ExtentTest>` variable to guarantee thread safety during parallel execution.

```java
import com.aventstack.extentreports.ExtentReports;
import com.aventstack.extentreports.ExtentTest;
import com.aventstack.extentreports.Status;
import org.testng.ITestContext;
import org.testng.ITestListener;
import org.testng.ITestResult;
public class ExtentTestListener implements ITestListener {
    // Initialize the main engine
    private static ExtentReports extent = ExtentManager.getInstance();
    // Create a ThreadLocal wrapper for parallel execution
    private static ThreadLocal<ExtentTest> extentTest = new ThreadLocal<>();
    @Override
    public void onTestStart(ITestResult result) {
        // Automatically create a new Test entry in the HTML report when a TestNG method starts
        ExtentTest test = extent.createTest(result.getMethod().getMethodName());
        extentTest.set(test);
    }
    @Override
    public void onTestSuccess(ITestResult result) {
        extentTest.get().log(Status.PASS, "Test Passed Successfully!");
    }
    @Override
    public void onTestFailure(ITestResult result) {
        // Log the failure and print the stack trace into the HTML report
        extentTest.get().log(Status.FAIL, "Test Failed: " + result.getThrowable());
        // (Optional: You could cast your WebDriver here and attach a Base64 screenshot!)
        // String base64Screenshot = ((TakesScreenshot)driver).getScreenshotAs(OutputType.BASE64);
        // extentTest.get().addScreenCaptureFromBase64String(base64Screenshot, "Failure Evidence");
    }
    @Override
    public void onTestSkipped(ITestResult result) {
        extentTest.get().log(Status.SKIP, "Test Skipped.");
    }
    @Override
    public void onFinish(ITestContext context) {
        // CRITICAL: You must call flush() at the very end of the suite, or the HTML file will be empty!
        extent.flush();
    }
}
```

---

## 5. Registering the Listener

Finally, register your new listener in your `testng.xml` file:

```xml
<!DOCTYPE suite SYSTEM "https://testng.org/testng-1.0.dtd" >
<suite name="Extent Regression Suite">
    <listeners>
        <listener class-name="com.mycodeyatra.listeners.ExtentTestListener" />
    </listeners>
    <test name="UI Tests">
        <classes>
            <class name="com.mycodeyatra.tests.LoginTest" />
        </classes>
    </test>
</suite>
```

When you run your suite, Extent Reports will magically track every test. When the execution finishes, open `target/ExtentReport.html` in any browser.

You will instantly see a stunning Dark Mode dashboard featuring interactive pie charts, system environment data, and a perfectly categorized list of your passing and failing tests—all contained within a single HTML file!

## Conclusion

Both Allure and Extent Reports are incredible tools.

If you are running your tests via a dedicated CI/CD pipeline like Jenkins, **Allure** is generally preferred because of its deep plugin integrations and historical trend tracking.

However, if you are running tests locally, executing them from a Docker container, or need to email the results immediately to a stakeholder without requiring them to install command-line tools, the single-page HTML portability of **Extent Reports** is absolutely unbeatable!

Now that we know how to generate beautiful dashboards, how do we decide *what* data actually goes on them? In our next tutorial, we will explore **Dashboard Metrics**, discussing the KPIs and statistical data points that engineering leaders actually care about.
