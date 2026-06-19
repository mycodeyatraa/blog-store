---
title: Beating the Flakiness: Mastering Baseline Management in Visual Tests
date: 27-Jun-2025
lastUpdated: 27-Jun-2025
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [java, selenium, visual-testing, baseline-management, test-automation, ashot, ui]
category: Selenium Java
categories: [Selenium Java, Visual Testing]
excerpt: >-
  Stop your visual tests from failing due to dynamic content. Learn how to use coordinate masking, DOM manipulation, and Git branch versioning to stabilize your Selenium visual regression suites.
readTime: 6 min read
---

# Beating the Flakiness: Mastering Baseline Management in Visual Tests

In our previous tutorial, we successfully implemented a pixel-to-pixel visual regression test using Selenium and the open-source `AShot` library. 

While it worked perfectly for a static homepage, real-world enterprise applications are almost never static. They contain live clocks, rotating ad banners, dynamic user profiles, and varying content layouts. 

If you use a strict pixel-to-pixel comparison engine on a dynamic website, your test will fail 100% of the time. In this tutorial, we will learn advanced **Baseline Management** strategies to stabilize our visual tests and eliminate false positives!

---

## 1. The Core Challenge: Dynamic Content

Let's assume our dashboard has a timestamp widget that constantly updates:

```html
<div id="welcome-message">Welcome back, Admin.</div>
<div id="live-clock">14:32:05 PST</div>
```

The first time we run our `AShot` test, the baseline image saves the clock rendering `14:32:05`. 
When the CI/CD pipeline runs the test again tomorrow, the clock will render `09:15:00`.

The pixel comparison engine will detect that the pixels forming the numbers have changed, and it will immediately throw a **Visual Regression Failure**.

This is called a "False Positive". The application isn't broken; the content just changed dynamically.

---

## 2. Strategy 1: Ignored Regions (Masking)

The most robust way to handle dynamic content is to "mask" it. We explicitly tell our visual testing engine to completely ignore the pixels inside a specific bounding box.

The `AShot` library natively supports adding ignored areas! Here is how we implement it:

```java
import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.chrome.ChromeDriver;
import ru.yandex.qatools.ashot.AShot;
import ru.yandex.qatools.ashot.Screenshot;
import ru.yandex.qatools.ashot.comparison.ImageDiff;
import ru.yandex.qatools.ashot.comparison.ImageDiffer;
import ru.yandex.qatools.ashot.coordinates.Coords;
import javax.imageio.ImageIO;
import java.awt.image.BufferedImage;
import java.io.File;
public class IgnoredRegionsTest {
    public static void main(String[] args) throws Exception {
        WebDriver driver = new ChromeDriver();
        driver.get("https://practice.mycodeyatra.com/dashboard");
        // Locate the dynamic element that causes flakiness
        WebElement liveClock = driver.findElement(By.id("live-clock"));
        // Add the element's coordinates to the AShot ignore list!
        Screenshot currentScreenshot = new AShot()
                .addIgnoredElement(By.id("live-clock"))
                .takeScreenshot(driver, liveClock); 
                // AShot will automatically calculate the bounding box and ignore it during comparison
        BufferedImage expectedImage = ImageIO.read(new File("src/test/resources/baselines/dashboard_baseline.png"));
        ImageDiffer imgDiff = new ImageDiffer();
        ImageDiff diff = imgDiff.makeDiff(expectedImage, currentScreenshot.getImage());
        if(diff.hasDiff()){
            System.out.println("❌ VISUAL REGRESSION DETECTED! (Ignoring the live clock)");
        } else {
            System.out.println("✅ Visual UI matches the baseline perfectly.");
        }
        driver.quit();
    }
}
```

By adding the ignored region, the ImageDiffer will literally skip the pixels occupied by the live clock, ensuring your test stays green!

---

## 3. Strategy 2: DOM Manipulation (Hiding Elements)

Sometimes, the dynamic content doesn't have a fixed bounding box. For example, a rotating ad banner might be a different size every time the page loads. In this case, `AShot`'s coordinate masking will fail because the coordinates keep changing.

Instead of masking the pixels, we can use Selenium's `JavascriptExecutor` to physically delete or hide the dynamic element from the DOM *before* we take the screenshot!

```java
import org.openqa.selenium.JavascriptExecutor;
public void hideDynamicAdBanner(WebDriver driver) {
    JavascriptExecutor js = (JavascriptExecutor) driver;
    // Completely hide the element from the rendering tree
    js.executeScript("document.getElementById('rotating-ad-banner').style.visibility = 'hidden';");
    // Alternatively, delete it entirely
    // js.executeScript("var element = document.getElementById('rotating-ad-banner'); element.parentNode.removeChild(element);");
}
@Test
public void testVisualRegressionWithoutAds() {
    driver.get("https://practice.mycodeyatra.com/dashboard");
    // Hide the dynamic content first!
    hideDynamicAdBanner(driver);
    // Now take the screenshot
    Screenshot currentScreenshot = new AShot().takeScreenshot(driver);
    // ... compare with baseline
}
```

---

## 4. Strategy 3: Baseline Versioning (Git Integration)

If a developer intentionally updates the application's CSS (e.g., changing the primary button color from Blue to Green), the visual test will legitimately fail. 

When this happens, you must update the Golden Baseline image.

**Best Practices for Baseline Storage:**
1. **Never store baselines in a separate database:** Your baseline images should live directly inside your Git repository (e.g., `src/test/resources/baselines/`).
2. **Tie baselines to the Branch:** When a developer creates a feature branch to update the UI, they should overwrite the baseline image in *their* specific branch.
3. **Code Review:** When they open a Pull Request, the reviewer can physically look at the image diff inside GitHub. If the new baseline image is correct, they merge the PR, instantly updating the baselines for the `main` branch pipeline!

## Conclusion

By implementing Ignored Regions via AShot, dynamically hiding elements via JavaScript, and storing your baseline images in Git, you can stabilize your visual regression suite and eliminate the vast majority of false positives.

However, writing masking coordinates and maintaining thousands of PNG images in a Git repo can get extremely tedious as your application scales. 

In our next tutorial, we will abandon open-source pixel matching entirely and step into the world of enterprise AI-driven visual testing with **Applitools Eyes**!
