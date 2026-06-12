---
title: Baseline Management: Stabilizing Visual Tests
date: 28-Sep-2024
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [selenium, java, visual-testing, ashot, baselines, aws-s3]
category: UI Automation
categories: [UI Automation, Selenium, Java]
excerpt: >-
  Eliminate visual test flakiness. Learn how to configure AShot to ignore dynamic WebElements and design enterprise storage strategies using AWS S3 for your baseline images.
readTime: 5 min read
---

# Baseline Management: Stabilizing Visual Tests

In our previous article, we successfully implemented pixel-by-pixel visual regression testing using AShot. However, if you run that test against a modern, dynamic web application, you will immediately encounter a massive problem: **Flakiness**.

If your dashboard contains a "Last Login" timestamp, or an ad banner that rotates images on every refresh, the pixels in that specific area will *always* be different from your baseline image. 

In this article, we will learn how to configure our visual tests to explicitly ignore dynamic elements, and discuss enterprise strategies for storing hundreds of baseline images.

---

## 1. Handling Dynamic Elements with AShot

To stop a visual test from failing due to dynamic data, we must tell the `ImageDiffer` engine to "blindfold" itself when looking at specific parts of the screen.

We do this by finding the dynamic WebElements via Selenium locators, extracting their coordinates, and passing those coordinates to AShot as **Ignored Areas**.

```java
package com.mycodeyatra.visual;
 
import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
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
 
public class DynamicVisualTest {
 
    @Test
    public void verifyDashboardWithIgnoredElements() throws Exception {
        WebDriver driver = ZapProxyManager.getProxiedDriver(); 
        driver.get("https://mycodeyatra.com/dashboard");
 
        // 1. Identify the dynamic element (e.g., a live clock or rotating ad)
        WebElement liveClock = driver.findElement(By.id("server-time-widget"));
 
        // 2. Capture the actual screenshot, explicitly ignoring the dynamic element
        Screenshot actualScreenshot = new AShot()
            .shootingStrategy(ShootingStrategies.viewportPasting(1000))
            .addIgnoredElement(By.id("server-time-widget"))
            .takeScreenshot(driver);
 
        // 3. Load the baseline
        BufferedImage expectedImage = ImageIO.read(new File("src/test/resources/baselines/dashboard.png"));
 
        // 4. Compare images using a configured ImageDiffer
        ImageDiffer imgDiff = new ImageDiffer();
        ImageDiff diff = imgDiff.makeDiff(expectedImage, actualScreenshot.getImage());
 
        if (diff.hasDiff()) {
            ImageIO.write(diff.getMarkedImage(), "PNG", new File("target/dashboard_diff.png"));
            Assert.fail("Visual Regression found outside of the ignored zones!");
        }
    }
}
```

When AShot generates the Diff Image upon a failure, the ignored `server-time-widget` will be blacked out or greyed out, proving that the engine completely skipped comparing those pixels.

---

## 2. Managing Baseline Storage

As your test suite grows, you will eventually have hundreds of baseline images representing different pages, modals, and hover states. Where should you store these assets?

### Approach A: Git Repository (Small Scale)
If you have fewer than 100 baseline images, you can store them directly in `src/test/resources/baselines`. 
- **Pros:** Version-controlled alongside your code.
- **Cons:** Git is notoriously bad at handling large binary files. As baselines are updated and overwritten, your `.git` folder will bloat massively, slowing down clones and CI pipelines.

### Approach B: Git LFS (Medium Scale)
Git Large File Storage (LFS) replaces large files such as audio samples, videos, datasets, and graphics with text pointers inside Git, while storing the file contents on a remote server.
- **Pros:** Keeps the repository lightweight while maintaining version control syntax.
- **Cons:** Requires explicit developer setup (`git lfs install`).

### Approach C: Cloud Storage (Enterprise Scale)
For true enterprise automation, baseline images are pushed to an AWS S3 Bucket or Azure Blob Storage.
When the test suite runs in Jenkins, the setup script uses the AWS SDK to download the `master` baselines into the local workspace before executing the tests.
- **Pros:** Infinite scaling, no Git bloat, easy to share across multiple different repositories.
- **Cons:** Requires writing cloud integration code and managing IAM roles.

---

## System Architecture

Here is the data flow for an Enterprise Visual Testing architecture using Cloud Storage:

![diagram_1](https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/posts/baseline-management-stabilizing-visual-tests/images/diagram_1.png)

## Conclusion

By configuring AShot to ignore dynamic WebElements, we completely eliminate the flakiness that plagues open-source visual testing. Furthermore, by moving our baseline storage out of Git and into AWS S3, we keep our test repository lightning-fast.

However, writing code to ignore every single dynamic element manually is tedious. What if an AI could just *look* at the screen and automatically know which parts to ignore? 

In our next article, we will graduate to commercial AI-driven visual testing by integrating **Applitools Eyes** into our Java framework!
