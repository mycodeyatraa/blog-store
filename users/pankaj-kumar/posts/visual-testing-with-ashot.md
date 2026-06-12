---
title: Visual Testing: AShot library screenshot comparison
date: 25-Sep-2024
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [selenium, java, visual-testing, ashot, screenshot, ui-automation]
category: UI Automation
categories: [UI Automation, Selenium, Java]
excerpt: >-
  Go beyond functional assertions. Learn how to perform pixel-by-pixel visual regression testing in Java using the open-source AShot library.
readTime: 5 min read
---

# Introduction to Visual Testing with AShot

For the last 48 articles, we have written hundreds of assertions like `Assert.assertEquals(button.getText(), "Login")` and `Assert.assertTrue(button.isDisplayed())`. 

But here is the hard truth: **Functional assertions cannot see.** 

If a developer accidentally pushes a CSS change that turns your "Login" button text white on a white background, or if a floating chat widget completely overlaps the checkout button, your Selenium test will still pass! Selenium sees the DOM, not the screen.

To guarantee pixel-perfect UIs, we must enter **Phase 8: Advanced Visual Testing**.

---

## 1. What is Visual Testing?

Visual Testing (or Visual Regression Testing) works by taking a screenshot of your web application in a known good state (the **Baseline**). During future test runs, the script takes a new screenshot (the **Actual**) and mathematically compares every single pixel between the two. 

If there is a mismatch (e.g., a button shifted 5 pixels to the left, or a font color changed), the test fails and generates a **Diff Image** highlighting the exact pixels that changed.

---

## 2. Setting up AShot

While there are paid enterprise tools for Visual Testing (which we will cover later), you can achieve pixel-perfect comparisons for free using an open-source Java library called **AShot**.

Add AShot to your `pom.xml`:

```xml
<dependency>
    <groupId>ru.yandex.qatools.ashot</groupId>
    <artifactId>ashot</artifactId>
    <version>1.5.4</version>
</dependency>
```

---

## 3. Capturing Full-Page Screenshots

By default, Selenium's `TakesScreenshot` interface only captures what is currently visible in the browser viewport. If your page requires scrolling, the bottom half gets cut off.

AShot solves this using a **Shooting Strategy** that scrolls the page automatically, taking multiple screenshots and stitching them together into one massive image.

```java
package com.mycodeyatra.visual;
 
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import ru.yandex.qatools.ashot.AShot;
import ru.yandex.qatools.ashot.Screenshot;
import ru.yandex.qatools.ashot.shooting.ShootingStrategies;
 
import javax.imageio.ImageIO;
import java.io.File;
 
public class BaselineGenerator {
 
    public static void captureBaseline() throws Exception {
        WebDriver driver = new ChromeDriver();
        driver.get("https://mycodeyatra.com/dashboard");
 
        // Take a full page screenshot by scrolling every 1000ms
        Screenshot baseline = new AShot()
            .shootingStrategy(ShootingStrategies.viewportPasting(1000))
            .takeScreenshot(driver);
 
        // Save the baseline image to the project directory
        ImageIO.write(baseline.getImage(), "PNG", new File("src/test/resources/baselines/dashboard.png"));
 
        driver.quit();
    }
}
```

---

## 4. Performing Pixel-by-Pixel Comparison

Once you have your baseline image stored in `src/test/resources`, you can write a TestNG assertion that compares the live application against it.

```java
package com.mycodeyatra.visual;
 
import org.openqa.selenium.WebDriver;
import org.testng.Assert;
import org.testng.annotations.Test;
import ru.yandex.qatools.ashot.AShot;
import ru.yandex.qatools.ashot.Screenshot;
import ru.yandex.qatools.ashot.comparison.ImageDiff;
import ru.yandex.qatools.ashot.comparison.ImageDiffer;
import ru.yandex.qatools.ashot.shooting.ShootingStrategies;
 
import javax.imageio.ImageIO;
import java.awt.image.BufferedImage;
import java.io.File;
 
public class VisualRegressionTest {
 
    @Test
    public void verifyDashboardLayout() throws Exception {
        WebDriver driver = ZapProxyManager.getProxiedDriver(); // Assume driver setup
        driver.get("https://mycodeyatra.com/dashboard");
 
        // 1. Capture the actual live screenshot
        Screenshot actualScreenshot = new AShot()
            .shootingStrategy(ShootingStrategies.viewportPasting(1000))
            .takeScreenshot(driver);
 
        // 2. Load the baseline image from disk
        BufferedImage expectedImage = ImageIO.read(new File("src/test/resources/baselines/dashboard.png"));
 
        // 3. Compare the images using ImageDiffer
        ImageDiffer imgDiff = new ImageDiffer();
        ImageDiff diff = imgDiff.makeDiff(expectedImage, actualScreenshot.getImage());
 
        // 4. If there is a difference, save the Diff Image highlighting the errors
        if (diff.hasDiff()) {
            File diffFile = new File("target/visual-diffs/dashboard_diff.png");
            ImageIO.write(diff.getMarkedImage(), "PNG", diffFile);
 
            Assert.fail("VISUAL REGRESSION DETECTED! Differences found. Check diff image at: " + diffFile.getAbsolutePath());
        }
    }
}
```

---

## System Architecture

Here is the exact workflow of a Visual Regression Test:

![diagram_1](https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/posts/visual-testing-with-ashot/images/diagram_1.png)

## Conclusion

Visual Testing acts as an incredibly powerful safety net. Instead of writing 50 different assertions to check the alignment and color of every element on the dashboard, a single visual assertion guarantees the entire page looks exactly as the designer intended.

However, strict pixel-by-pixel comparison has a major flaw: **Flakiness**. If an image carousal loads a different picture, or a timestamp updates on the screen, the test will fail!

In our next article, we will tackle **Baseline Management** and learn how to configure AShot to explicitly ignore dynamic elements (like dates and changing text) to create stable, rock-solid visual tests.
