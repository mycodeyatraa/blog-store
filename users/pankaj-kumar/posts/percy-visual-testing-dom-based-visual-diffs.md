---
title: Percy Visual Testing: DOM-based Visual Diffs
date: 12-Oct-2024
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [selenium, java, visual-testing, percy, browserstack, ui-automation]
category: UI Automation
categories: [UI Automation, Selenium, Java]
excerpt: >-
  Explore DOM-based visual testing. Learn how to integrate the Percy Java SDK to extract application assets and generate lightning-fast, highly accurate visual diffs in the cloud.
readTime: 5 min read
---

# Percy Visual Testing: DOM-based Visual Diffs

Throughout Phase 8, we have explored multiple ways to visually test our application. We started with the open-source **AShot** (which relies on strict pixel matching), and then graduated to **Applitools Eyes** (which uses AI to understand layout and content).

In this final article of our Visual Testing series, we will explore a powerful third alternative: **Percy** (acquired by BrowserStack). Percy takes a fundamentally different architectural approach to visual testing compared to traditional screenshot tools, making it one of the fastest and most reliable tools on the market.

---

## 1. How Percy is Different

Most visual testing tools follow a simple pattern: your Selenium script instructs the local browser to take a screenshot, and that PNG file is uploaded to the cloud for comparison. 

**Percy does not upload screenshots.** 

When you call the Percy SDK, it freezes the page and extracts the raw **DOM snapshot** along with all associated CSS, images, and fonts. It uploads this raw data to the Percy Cloud. Percy's servers then reconstruct your web page in their own highly optimized browsers and take the screenshots there.

Because Percy handles the rendering internally, you don't need to worry about local GPU differences, font rendering artifacts, or taking time to resize your local browser window.

---

## 2. Setting Up the Percy Java SDK

To use Percy, you need to sign up for a free account, obtain your `PERCY_TOKEN`, and add the Java SDK to your `pom.xml`:

```xml
<dependency>
    <groupId>io.percy</groupId>
    <artifactId>percy-java-selenium</artifactId>
    <version>1.2.0</version>
</dependency>
```

Because Percy intercepts network traffic to extract assets, you must run your Java tests using the Percy CLI wrapper. You install the CLI via Node.js:
```bash
npm install -g @percy/cli
```

---

## 3. Writing Your First Percy Test

Integrating Percy into your existing Selenium framework requires literally one line of code: `percy.snapshot()`.

```java
package com.mycodeyatra.visual;
 
import io.percy.selenium.Percy;
import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.testng.annotations.AfterMethod;
import org.testng.annotations.BeforeMethod;
import org.testng.annotations.Test;
 
import java.util.Arrays;
 
public class PercyVisualTest {
 
    private WebDriver driver;
    private Percy percy;
 
    @BeforeMethod
    public void setup() {
        driver = new ChromeDriver();
 
        // Initialize the Percy SDK with your WebDriver
        percy = new Percy(driver);
    }
 
    @Test
    public void verifyCheckoutPage() {
        driver.get("https://mycodeyatra.com/checkout");
 
        // Perform some functional actions
        driver.findElement(By.id("cart-btn")).click();
 
        // Take a visual snapshot!
        // We can pass an array of widths to instantly render this DOM on mobile, tablet, and desktop
        percy.snapshot("Checkout Modal View", Arrays.asList(375, 768, 1440));
    }
 
    @AfterMethod
    public void teardown() {
        driver.quit();
    }
}
```

To run this test, you set your token and execute Maven through the Percy wrapper:
```bash
export PERCY_TOKEN=your_token_here
percy exec -- mvn clean test -Dtest=PercyVisualTest
```

---

## 4. The Percy Dashboard Workflow

Once the test finishes, your terminal will print a link to the Percy Dashboard. 

Percy integrates deeply with GitHub/GitLab Pull Requests. When a developer submits a PR, Percy renders the DOM snapshots and compares them against the `master` branch baselines.

If visual differences are found, the PR is marked as **"Blocked"**. A QA engineer clicks the link, reviews the visual diffs (highlighted in red), and clicks **"Approve"** if the changes were intentional (like a planned redesign) or **"Request Changes"** if the changes are a bug.

---

## System Architecture

Here is the data flow for DOM-based visual testing:

![diagram_1](https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/posts/percy-visual-testing-dom-based-visual-diffs/images/diagram_1.png)

## Conclusion

Both Percy and Applitools are incredible enterprise platforms, but they serve different architectural philosophies. Applitools relies heavily on Artificial Intelligence to process images uploaded from your local machine, while Percy relies on blazing-fast DOM extraction and rendering the site on their own server infrastructure.

This brings us to the end of **Phase 8: Advanced Visual Testing**! You now know how to stabilize pixel matching with AShot, emulate mobile devices, leverage AI with Applitools, and perform DOM-based diffs with Percy.

Next, we will move into **Phase 9: Accessibility**, where we will learn how to integrate Axe-Core to ensure our applications are usable by everyone, including users with disabilities!
