---
title: Testing on the Go: Mobile Web Emulation with Selenium Java
date: 03-Jul-2025
lastUpdated: 03-Jul-2025
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [java, selenium, visual-testing, mobile-emulation, chrome-devtools, responsive, test-automation]
category: Selenium Java
categories: [Selenium Java, Visual Testing]
excerpt: >-
  Resizing your browser isn't enough to test mobile sites. Learn how to configure ChromeOptions to leverage Chrome DevTools Mobile Emulation, spoofing User-Agents and physical device metrics natively in Selenium.
readTime: 6 min read
---

# Testing on the Go: Mobile Web Emulation with Selenium Java

In our previous tutorial, we explored how to test responsive web layouts by resizing the browser viewport natively and via the Applitools Ultrafast Grid.

However, resizing a desktop browser window to 375x812 pixels does **not** perfectly replicate a true mobile device. A desktop browser resized to mobile dimensions still reports itself as a desktop browser to the backend server. It still uses a mouse instead of a touch screen. 

If your web application contains specific logic like `if (isMobileDevice) { showMobileAppDownloadBanner(); }`, simply resizing the browser will not trigger that logic. 

To fully test the mobile web experience without buying thousands of dollars of real physical phones, we need to use **Chrome DevTools Mobile Emulation**.

---

## 1. What is Mobile Emulation?

Mobile Emulation is a feature built directly into Google Chrome. When activated, Chrome overrides its User-Agent string, sets the physical screen dimensions, adjusts the device pixel ratio (for retina displays), and changes mouse clicks into touch events.

When Selenium drives an emulated Chrome session, the web server genuinely believes it is talking to a physical iPhone or Pixel device!

---

## 2. Configuring Mobile Emulation in Selenium

We can configure Mobile Emulation using the `ChromeOptions` class. We simply pass a dictionary mapping containing the `deviceName` we wish to emulate.

```java
import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.chrome.ChromeOptions;
import org.testng.Assert;
import org.testng.annotations.Test;
import java.util.HashMap;
import java.util.Map;
public class MobileEmulationTest {
    @Test
    public void testIPhone12ProEmulation() {
        // 1. Create a map to hold the emulation settings
        Map<String, String> mobileEmulation = new HashMap<>();
        // 2. Specify the exact device to emulate
        // This must match exactly with a device name found in Chrome DevTools
        mobileEmulation.put("deviceName", "iPhone 12 Pro");
        // 3. Add the emulation settings to ChromeOptions
        ChromeOptions options = new ChromeOptions();
        options.setExperimentalOption("mobileEmulation", mobileEmulation);
        // 4. Initialize WebDriver with the mobile options
        WebDriver driver = new ChromeDriver(options);
        // 5. Navigate to the application
        driver.get("https://practice.mycodeyatra.com/");
        // 6. Assert that the Mobile-specific UI elements are rendered
        boolean isMobileMenuVisible = driver.findElement(By.id("mobile-hamburger-menu")).isDisplayed();
        Assert.assertTrue(isMobileMenuVisible, "Mobile hamburger menu is missing on iPhone 12 Pro!");
        // Assert that Desktop elements are physically hidden
        boolean isDesktopNavVisible = driver.findElement(By.id("desktop-nav-links")).isDisplayed();
        Assert.assertFalse(isDesktopNavVisible, "Desktop navigation should not be visible on mobile!");
        driver.quit();
    }
}
```

---

## 3. Custom Device Emulation

What if you need to test a custom Android tablet that isn't listed in the default Chrome DevTools device list? 

Instead of passing a `deviceName`, you can explicitly define the metrics: User-Agent, Width, Height, and Pixel Ratio.

```java
@Test
public void testCustomTabletEmulation() {
    Map<String, Object> deviceMetrics = new HashMap<>();
    deviceMetrics.put("width", 800);
    deviceMetrics.put("height", 1280);
    deviceMetrics.put("pixelRatio", 2.0);
    deviceMetrics.put("touch", true); // Enable Touch Events!
    Map<String, Object> mobileEmulation = new HashMap<>();
    mobileEmulation.put("deviceMetrics", deviceMetrics);
    // Override the User-Agent to simulate a custom Android Tablet
    mobileEmulation.put("userAgent", "Mozilla/5.0 (Linux; Android 11; Custom Tablet) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/90.0.4430.91 Mobile Safari/537.36");
    ChromeOptions options = new ChromeOptions();
    options.setExperimentalOption("mobileEmulation", mobileEmulation);
    WebDriver driver = new ChromeDriver(options);
    driver.get("https://practice.mycodeyatra.com/");
    System.out.println("Current User Agent: " + 
        ((org.openqa.selenium.JavascriptExecutor)driver).executeScript("return navigator.userAgent;"));
    driver.quit();
}
```

---

## 4. Why Emulation isn't "Real" Mobile Testing

While Chrome DevTools Mobile Emulation is fantastic for fast, localized responsive UI testing, it is important to remember its limitations:

1. **It is still Chrome:** Emulating an iPhone in Chrome does *not* run the Apple WebKit engine. It runs the Chrome Blink engine. If there is a Safari-specific rendering bug, Mobile Emulation will not catch it.
2. **Hardware Constraints:** Emulation does not simulate the CPU or Memory constraints of a physical mobile device. A heavily bloated JavaScript application might run perfectly in Emulation on your powerful Macbook, but crash completely on a real 5-year-old Android phone.

If you need 100% accurate mobile testing, you must upgrade from Selenium to **Appium**, which drives actual native mobile browsers on physical devices or emulators.

## Conclusion

Mobile Emulation bridges the gap between Desktop Automation and Real Device Testing. By simply injecting `mobileEmulation` into your `ChromeOptions`, you can run a massive suite of mobile web UI tests directly on your CI/CD pipeline in seconds, without paying for expensive cloud device farms!

This concludes **Phase 8: Visual Testing**. In our next curriculum phase, we will shift gears completely to focus on ensuring our applications are accessible to everyone: **Phase 9 - Accessibility Testing (a11y)**!
