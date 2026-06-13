---
title: Responsive Testing: Validating Mobile and Tablet Viewports
date: 05-Oct-2024
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [selenium, java, visual-testing, responsive-design, applitools, mobile-testing]
category: UI Automation
categories: [UI Automation, Selenium, Java]
excerpt: >-
  Guarantee perfect UIs across all devices. Learn how to programmatically resize the Selenium viewport and execute visual tests concurrently across mobile and desktop profiles using Applitools Ultrafast Grid.
readTime: 5 min read
---

# Responsive Testing: Validating Mobile and Tablet Viewports

Modern web applications are built with responsive design. A "Hamburger Menu" might only appear when the screen is less than 768 pixels wide, while a multi-column dashboard might collapse into a single vertical scroll on a mobile device.

In our previous articles, we learned how to perform visual regression testing using AShot and Applitools. However, if you only run those tests maximized on a 1080p monitor, you are completely blind to bugs that only occur on iPads and iPhones.

In this article, we will learn how to automate Responsive Testing by manipulating the browser viewport and running visual validations across multiple breakpoints!

---

## 1. Understanding Breakpoints

A "breakpoint" is a specific screen width where the layout of the website radically changes (driven by CSS Media Queries).

The most common industry breakpoints are:
- **Mobile (Portrait):** 375 x 667 (iPhone SE)
- **Mobile (Landscape) / Small Tablet:** 768 x 1024 (iPad Portrait)
- **Desktop:** 1440 x 900
- **Large Desktop:** 1920 x 1080

If we want to guarantee our application works perfectly for all users, our visual tests must capture baselines for *every single one of these resolutions*.

---

## 2. Resizing the Viewport in Selenium

Selenium provides a very straightforward API to control the size of the physical browser window using the `Dimension` class.

Here is a simple TestNG test that iterates through an array of breakpoints, resizes the browser, and asserts that a responsive element (like the Hamburger Menu) appears correctly.

```java
package com.mycodeyatra.visual;
 
import org.openqa.selenium.By;
import org.openqa.selenium.Dimension;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.chrome.ChromeDriver;
import org.testng.Assert;
import org.testng.annotations.AfterMethod;
import org.testng.annotations.BeforeMethod;
import org.testng.annotations.Test;
 
public class ResponsiveTest {
 
    private WebDriver driver;
 
    @BeforeMethod
    public void setup() {
        driver = new ChromeDriver();
        driver.get("https://mycodeyatra.com");
    }
 
    @Test
    public void testHamburgerMenuAppearsOnMobile() throws InterruptedException {
        // 1. Set to Desktop (1920x1080)
        driver.manage().window().setSize(new Dimension(1920, 1080));
        Thread.sleep(1000); // Wait for CSS transitions
 
        WebElement hamburgerMenu = driver.findElement(By.id("mobile-nav-toggle"));
        Assert.assertFalse(hamburgerMenu.isDisplayed(), "Hamburger menu should NOT be visible on Desktop!");
 
        // 2. Set to Mobile (375x667)
        driver.manage().window().setSize(new Dimension(375, 667));
        Thread.sleep(1000); // Wait for CSS transitions
 
        Assert.assertTrue(hamburgerMenu.isDisplayed(), "Hamburger menu MUST be visible on Mobile!");
    }
 
    @AfterMethod
    public void teardown() {
        driver.quit();
    }
}
```

---

## 3. Combining Resizing with Applitools Eyes

Functional assertions (like checking if the hamburger menu is visible) are a great start, but they don't prove the layout actually looks good. 

To achieve true responsive coverage, we should integrate our viewport resizing loop with **Applitools Eyes**. 

Applitools makes this incredibly easy with its **Ultrafast Grid**. Instead of resizing the local Chrome browser slowly, we can instruct Applitools to take one snapshot, upload the DOM, and render it simultaneously across dozens of device profiles in the cloud!

```java
package com.mycodeyatra.visual;
 
import com.applitools.eyes.selenium.BrowserType;
import com.applitools.eyes.selenium.Configuration;
import com.applitools.eyes.selenium.Eyes;
import com.applitools.eyes.visualgrid.model.DeviceName;
import com.applitools.eyes.visualgrid.model.ScreenOrientation;
import com.applitools.eyes.visualgrid.services.VisualGridRunner;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.testng.annotations.Test;
 
public class CrossDeviceVisualTest {
 
    @Test
    public void runUltrafastGrid() {
        WebDriver driver = new ChromeDriver();
 
        // 1. Initialize the Visual Grid Runner (Cloud Execution)
        VisualGridRunner runner = new VisualGridRunner(10); // 10 parallel threads
        Eyes eyes = new Eyes(runner);
 
        // 2. Configure multiple viewports
        Configuration config = new Configuration();
        config.setApiKey(System.getenv("APPLITOOLS_API_KEY"));
 
        // Add Desktop Browsers
        config.addBrowser(1920, 1080, BrowserType.CHROME);
        config.addBrowser(1440, 900, BrowserType.FIREFOX);
 
        // Add Mobile Emulation
        config.addDeviceEmulation(DeviceName.iPhone_X, ScreenOrientation.PORTRAIT);
        config.addDeviceEmulation(DeviceName.iPad, ScreenOrientation.LANDSCAPE);
 
        eyes.setConfiguration(config);
 
        // 3. Run the test
        driver.get("https://mycodeyatra.com/dashboard");
 
        eyes.open(driver, "MyCodeYatra App", "Responsive Dashboard Test");
        eyes.checkWindow("Dashboard Main View");
        eyes.closeAsync(); // Fire and forget to the cloud
 
        driver.quit();
 
        // 4. Wait for all 4 cloud environments to finish analyzing
        runner.getAllTestResults();
    }
}
```

---

## System Architecture

Here is the difference between local resizing and cloud-rendered visual grids:

![diagram_1](https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/posts/responsive-testing-mobile-tablet-viewport/images/diagram_1.png)

## Conclusion

Responsive design testing is no longer optional. By programmatically altering the `Dimension` of your WebDriver, you can easily simulate smaller screens to trigger mobile-specific CSS. 

For enterprise-scale teams, combining this concept with a cloud rendering engine like Applitools Ultrafast Grid allows you to visually validate dozens of device configurations in seconds without ever having to manage complex Selenium Grid infrastructure.

In our next article, we will take mobile testing a step further by using **Chrome DevTools Device Emulation**, which not only changes the screen size but actually spoofs the browser's User-Agent string to fool the backend into thinking you are on a real iPhone!
