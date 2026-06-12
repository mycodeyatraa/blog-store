---
title: Applitools Eyes: AI visual validation with Java SDK
date: 01-Oct-2024
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [selenium, java, visual-testing, applitools, ai-testing, ui-automation]
category: UI Automation
categories: [UI Automation, Selenium, Java]
excerpt: >-
  Ditch pixel matching for AI. Learn how to integrate Applitools Eyes into your Selenium Java framework to perform intelligent, layout-aware visual regression testing.
readTime: 5 min read
---

# Applitools Eyes: AI Visual Validation

In our previous articles, we used AShot to perform pixel-by-pixel comparisons. While effective for simple pages, strict pixel matching breaks down completely in modern enterprise applications. 

If your application shifts a button by 2 pixels to the right, AShot will instantly fail the test. But a human user wouldn't even notice the change. What we need is a tool that "sees" the application the way a human does. 

Enter **Applitools Eyes**, the industry leader in Visual AI testing.

---

## 1. What is Visual AI?

Unlike open-source tools that mathematically compare colors and coordinates, Applitools uses Artificial Intelligence to analyze the layout, structure, and content of a web page. 

It understands that a table is a table, and a paragraph is a paragraph. If dynamic data (like a user's name or a daily changing image) loads on the screen, the AI can be configured to verify the *Layout* remains intact while completely ignoring the *Content*.

---

## 2. Setting Up the Applitools Java SDK

To integrate Applitools with your Selenium Java framework, you need to sign up for a free Applitools account to get an API Key, and then add their SDK to your `pom.xml`:

```xml
<dependency>
    <groupId>com.applitools</groupId>
    <artifactId>eyes-selenium-java5</artifactId>
    <version>5.53.0</version>
</dependency>
```

---

## 3. Writing Your First AI Visual Test

Applitools integrates seamlessly with Selenium. Instead of writing dozens of assertions to check if elements are displayed, we simply wrap our Selenium test in an `Eyes` validation block.

```java
package com.mycodeyatra.visual;
 
import com.applitools.eyes.selenium.Eyes;
import com.applitools.eyes.selenium.Configuration;
import com.applitools.eyes.MatchLevel;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.testng.annotations.AfterMethod;
import org.testng.annotations.BeforeMethod;
import org.testng.annotations.Test;
 
public class ApplitoolsVisualTest {
 
    private WebDriver driver;
    private Eyes eyes;
 
    @BeforeMethod
    public void setup() {
        driver = new ChromeDriver();
 
        // 1. Initialize the Eyes SDK
        eyes = new Eyes();
 
        // 2. Set your API Key (Always use environment variables!)
        Configuration config = new Configuration();
        config.setApiKey(System.getenv("APPLITOOLS_API_KEY"));
 
        // 3. Set the Match Level
        // STRICT = Human eye comparison
        // LAYOUT = Ignores text/colors, checks structure only (Great for dynamic data!)
        config.setMatchLevel(MatchLevel.STRICT);
 
        eyes.setConfiguration(config);
    }
 
    @Test
    public void verifyLoginDashboardLayout() {
        // 4. Open Applitools session
        eyes.open(driver, "MyCodeYatra App", "Dashboard Visual Test");
 
        driver.get("https://mycodeyatra.com/dashboard");
 
        // 5. Tell Applitools to take a snapshot and analyze it using AI
        eyes.checkWindow("Main Dashboard");
 
        // 6. Close the session (This automatically asserts if visual differences exist)
        eyes.close();
    }
 
    @AfterMethod
    public void teardown() {
        driver.quit();
        eyes.abortIfNotClosed(); // Failsafe
    }
}
```

---

## 4. The Magic of the Applitools Dashboard

When you run `eyes.close()`, the SDK uploads the snapshot to the Applitools cloud. 

If it is the very first time you are running the test, the image is automatically saved as the **Baseline**.

If it is a subsequent run, the AI compares the new snapshot against the baseline. If it detects a mismatch, the test fails, and you are provided a link to the Applitools Dashboard.

In the dashboard, you can:
- View the exact differences highlighted in pink.
- Draw **Ignore Regions** over highly dynamic elements (like a flashing ad banner).
- Click the "Thumbs Up" button to accept the new changes, instantly updating your baseline in the cloud!

---

## System Architecture

Here is the data flow of an Applitools test execution:

![diagram_1](https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/posts/applitools-eyes-ai-visual-validation/images/diagram_1.png)

## Conclusion

By swapping pixel-based comparison for AI-driven Layout and Strict matching, Applitools Eyes completely eradicates the flakiness of visual testing while completely removing the burden of managing baseline images in your local Git repository.

In our next article, we will explore another critical aspect of modern web design: **Responsive Testing**. We will learn how to visually validate our application across different Mobile and Tablet viewports!
