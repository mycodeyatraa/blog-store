---
title: Breaking the Grid: Responsive Visual Testing in Selenium Java
date: 01-Jul-2025
lastUpdated: 01-Jul-2025
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [java, selenium, visual-testing, responsive, applitools, grid, mobile, test-automation]
category: Selenium Java
categories: [Selenium Java, Visual Testing]
excerpt: >-
  Mobile screens require mobile layouts. Learn how to execute responsive visual tests by dynamically resizing the browser natively in Selenium, and how to scale execution using the Applitools Ultrafast Grid.
readTime: 6 min read
---

# Breaking the Grid: Responsive Visual Testing in Selenium Java

Modern websites are rarely built for a single screen size. They use responsive CSS frameworks (like Bootstrap or Tailwind) to dynamically collapse navigation menus into "hamburger" icons, stack side-by-side columns into single vertical feeds, and hide large background images when viewed on mobile devices.

Testing a website exclusively on a 1920x1080 desktop monitor is no longer sufficient. You must visually validate that the responsive breakpoints trigger correctly across Tablets and Mobile screens.

In this tutorial, we will learn how to orchestrate Responsive Visual Testing using Selenium Java, both natively and by supercharging it with the Applitools Ultrafast Grid!

---

## 1. The Challenge of Responsive Testing

To test responsive design natively, you must physically resize the browser window. 

When you resize the browser, the DOM physically changes. A `<nav id="desktop-menu">` might be destroyed, and a `<button id="mobile-hamburger">` might be dynamically injected.

If you are using open-source visual testing (like `AShot`), you have to manage a completely separate set of baseline images for every single screen size you test. 

- `homepage_desktop_baseline.png`
- `homepage_tablet_baseline.png`
- `homepage_mobile_baseline.png`

---

## 2. Native Responsive Testing via Selenium Dimension

If you want to manually test responsive breakpoints without cloud tools, you can use Selenium's native `manage().window().setSize()` method.

Here is how you write a test that dynamically iterates through standard viewport dimensions:

```java
import org.openqa.selenium.Dimension;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.testng.annotations.Test;
public class NativeResponsiveTest {
    @Test
    public void testResponsiveBreakpoints() {
        WebDriver driver = new ChromeDriver();
        driver.get("https://practice.mycodeyatra.com/responsive");
        // Define our target breakpoints
        Dimension[] viewports = {
            new Dimension(1920, 1080), // Desktop
            new Dimension(1024, 768),  // Tablet Landscape
            new Dimension(768, 1024),  // Tablet Portrait
            new Dimension(375, 812)    // iPhone X / Mobile
        };
        for (Dimension viewport : viewports) {
            // 1. Resize the browser
            driver.manage().window().setSize(viewport);
            // Allow CSS animations to settle
            try { Thread.sleep(1000); } catch (Exception e) {}
            System.out.println("Testing Viewport: " + viewport.getWidth() + "x" + viewport.getHeight());
            // 2. Perform Functional Assertions based on width
            if (viewport.getWidth() <= 768) {
                // Assert hamburger menu exists on Mobile/Tablet
                boolean hasHamburger = driver.findElements(By.id("hamburger-menu")).size() > 0;
                System.out.println("Hamburger Menu Present: " + hasHamburger);
            } else {
                // Assert standard navigation exists on Desktop
                boolean hasDesktopNav = driver.findElements(By.id("desktop-nav")).size() > 0;
                System.out.println("Desktop Nav Present: " + hasDesktopNav);
            }
            // 3. Optional: Integrate AShot here to take a screenshot and compare
            // against the specific baseline image for this viewport size!
        }
        driver.quit();
    }
}
```

This works, but it is incredibly slow. Resizing the browser, waiting for CSS reflows, and executing serial assertions across 4 different viewports turns a 5-second test into a 30-second test. Multiply that by 500 tests, and your pipeline slows to a crawl.

---

## 3. Scaling Responsive Testing with Applitools Ultrafast Grid

Instead of manually resizing your local browser, we can use the Applitools Ultrafast Grid.

With the Ultrafast Grid, you run your Selenium test **exactly once** on a standard Desktop browser. When you call `eyes.check()`, Applitools extracts the DOM, uploads it to the cloud, and renders it simultaneously across dozens of mobile and desktop configurations in parallel!

```java
import com.applitools.eyes.selenium.BrowserType;
import com.applitools.eyes.selenium.Configuration;
import com.applitools.eyes.selenium.Eyes;
import com.applitools.eyes.visualgrid.model.DeviceName;
import com.applitools.eyes.visualgrid.model.ScreenOrientation;
import com.applitools.eyes.visualgrid.services.VisualGridRunner;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.testng.annotations.Test;
public class ApplitoolsResponsiveTest {
    @Test
    public void testUltrafastGrid() {
        WebDriver driver = new ChromeDriver();
        // 1. Initialize the Visual Grid Runner (Concurrency = 5 parallel browsers)
        VisualGridRunner runner = new VisualGridRunner(5);
        Eyes eyes = new Eyes(runner);
        // 2. Create the Configuration
        Configuration config = new Configuration();
        config.setApiKey(System.getenv("APPLITOOLS_API_KEY"));
        config.setTestName("Responsive Homepage Layout");
        config.setAppName("MyCodeYatra App");
        // 3. Add Desktop Browsers
        config.addBrowser(1920, 1080, BrowserType.CHROME);
        config.addBrowser(1366, 768, BrowserType.FIREFOX);
        config.addBrowser(1024, 768, BrowserType.SAFARI);
        // 4. Add Mobile Emulation!
        config.addDeviceEmulation(DeviceName.iPhone_X, ScreenOrientation.PORTRAIT);
        config.addDeviceEmulation(DeviceName.Pixel_4, ScreenOrientation.LANDSCAPE);
        eyes.setConfiguration(config);
        // 5. Execute the Test!
        eyes.open(driver);
        driver.get("https://practice.mycodeyatra.com/");
        // This single line of code will take 5 different screenshots across 5 different environments!
        eyes.checkWindow("Responsive Homepage - Initial Load");
        driver.quit();
        eyes.closeAsync();
        // Wait for all the cloud rendering to finish
        runner.getAllTestResults(true);
    }
}
```

## Conclusion

Responsive testing is mandatory for any modern web application. Whether you choose to iterate natively using `driver.manage().window().setSize()` or leverage the massive parallelism of the Applitools Ultrafast Grid, ensuring your application looks perfect on every screen size is a critical skill for a QA Engineer.

In our next and final tutorial of the Visual Testing series, we will dive deeper into the world of Mobile, specifically exploring **Mobile Web Emulation** using Chrome DevTools!
