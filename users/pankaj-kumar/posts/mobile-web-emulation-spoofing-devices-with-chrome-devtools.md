---
title: Mobile Web Emulation: Spoofing Devices with Chrome DevTools
date: 08-Oct-2024
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [selenium, java, visual-testing, mobile-emulation, chrome-devtools, cdp]
category: UI Automation
categories: [UI Automation, Selenium, Java]
excerpt: >-
  Test the true mobile experience without a device farm. Learn how to spoof User-Agents, enable touch events, and simulate custom device metrics using Selenium and Chrome DevTools Protocol.
readTime: 5 min read
---

# Mobile Web Emulation: Spoofing Devices with Chrome DevTools

In our previous article on Responsive Testing, we successfully simulated mobile layouts by manually resizing the Selenium WebDriver window using `driver.manage().window().setSize()`. 

While this triggers CSS Media Queries (forcing menus to collapse and grids to stack), it has a fundamental flaw: **The backend still knows you are on a desktop.**

If your application serves completely different HTML for mobile users, or relies on Javascript Touch Events instead of Mouse Clicks, simply resizing the window won't test the actual mobile code path. 

In this article, we will learn how to use Chrome's built-in Mobile Emulation feature to spoof our User-Agent and device metrics, tricking the server into believing we are testing from a real iPhone!

---

## 1. Resizing vs. Emulation

Let's clearly define the difference between the two approaches:

- **Viewport Resizing:** Changes the width/height of the browser. Tests responsive CSS. The User-Agent remains `Windows NT 10.0; Win64; x64`.
- **Mobile Emulation:** Changes the width/height, scales the pixel ratio, enables Touch Events, and completely spoofs the User-Agent to `iPhone; CPU iPhone OS 13_2_3`. Tests the actual mobile application logic.

---

## 2. Enabling Mobile Emulation in ChromeOptions

Selenium provides a direct bridge to Chrome's DevTools emulation features via `ChromeOptions`. 

There are two ways to configure this: by explicitly providing device metrics (width, height, pixel ratio), or by simply providing the name of a known device (like "iPhone X").

Here is how you inject an iPhone X emulator into your WebDriver initialization:

```java
package com.mycodeyatra.visual;
 
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.chrome.ChromeOptions;
import org.testng.Assert;
import org.testng.annotations.Test;
 
import java.util.HashMap;
import java.util.Map;
 
public class MobileEmulationTest {
 
    @Test
    public void testApplicationAsIphoneX() {
        // 1. Create a map containing the emulation settings
        Map<String, String> mobileEmulation = new HashMap<>();
 
        // You can specify exact dimensions, or just use a known device name
        mobileEmulation.put("deviceName", "iPhone X");
 
        // 2. Pass the emulation map into ChromeOptions
        ChromeOptions options = new ChromeOptions();
        options.setExperimentalOption("mobileEmulation", mobileEmulation);
 
        // 3. Launch the browser
        WebDriver driver = new ChromeDriver(options);
 
        // 4. Navigate to the app
        driver.get("https://mycodeyatra.com");
 
        // 5. Verify the User-Agent was successfully spoofed!
        String userAgent = (String) ((org.openqa.selenium.JavascriptExecutor) driver)
                .executeScript("return navigator.userAgent;");
 
        System.out.println("Current User-Agent: " + userAgent);
        Assert.assertTrue(userAgent.contains("iPhone"), "Backend still thinks we are on Desktop!");
 
        driver.quit();
    }
}
```

---

## 3. Custom Device Emulation

What if you need to test a custom device that isn't in Chrome's default list, like a proprietary rugged tablet used in a warehouse? 

You can explicitly pass the `deviceMetrics` and the `userAgent` string yourself:

```java
Map<String, Object> deviceMetrics = new HashMap<>();
deviceMetrics.put("width", 800);
deviceMetrics.put("height", 1280);
deviceMetrics.put("pixelRatio", 1.5);
deviceMetrics.put("touch", true); // Enable touch events!
 
Map<String, Object> mobileEmulation = new HashMap<>();
mobileEmulation.put("deviceMetrics", deviceMetrics);
mobileEmulation.put("userAgent", "Mozilla/5.0 (Linux; Android 10; RuggedTablet X1) AppleWebKit/537.36");
 
ChromeOptions options = new ChromeOptions();
options.setExperimentalOption("mobileEmulation", mobileEmulation);
```

By enabling `touch: true`, Selenium's standard `click()` events will automatically be translated into touch events inside the browser, allowing you to test mobile-only Javascript sliders and swipe components!

---

## System Architecture

Here is the data flow showing how Mobile Emulation intercepts the network layer:

![diagram_1](https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/posts/mobile-web-emulation-spoofing-devices-with-chrome-devtools/images/diagram_1.png)

## Conclusion

Mobile Emulation is the absolute best way to test the mobile web experience without having to maintain a massive farm of physical devices or pay for expensive cloud device providers like BrowserStack. By spoofing the User-Agent and enabling touch events, you ensure both your frontend layout and your backend routing logic are thoroughly tested.

In our final article of Phase 8, we will explore one more enterprise visual testing tool: **Percy Visual Testing** (by BrowserStack), and see how it compares to Applitools!
