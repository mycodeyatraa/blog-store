---
title: Seeing Clearly: Automating Color Contrast Testing in Selenium
date: 13-Jul-2025
lastUpdated: 13-Jul-2025
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [java, selenium, accessibility, a11y, color-contrast, wcag, axe-core]
category: Selenium Java
categories: [Selenium Java, Accessibility Testing]
excerpt: >-
  Are your buttons readable? Learn the math behind WCAG 4.5:1 contrast ratios, and how to automate color contrast validation using Axe-Core and native Selenium CSS extractions.
readTime: 6 min read
---

# Seeing Clearly: Automating Color Contrast Testing in Selenium

A beautiful website is useless if your users cannot read the text. For individuals with low vision, color blindness, or simply someone trying to read their phone in bright sunlight, poor color contrast is one of the most frustrating barriers on the internet.

The WCAG (Web Content Accessibility Guidelines) explicitly mandates strict mathematical ratios between the foreground (text) color and the background color. 

In this tutorial, we will learn exactly how WCAG calculates contrast, how Axe-Core automates it, and how to write custom Selenium Java logic to manually validate CSS color contrast!

---

## 1. The WCAG Color Contrast Math

WCAG defines contrast as a ratio ranging from **1:1** (white text on a white background) to **21:1** (black text on a white background).

The WCAG 2.1 AA standard requires:
- **Normal Text** (below 18pt or 14pt bold) must have a contrast ratio of at least **4.5:1**.
- **Large Text** (18pt and above, or 14pt bold and above) must have a contrast ratio of at least **3.0:1**.
- **UI Components** (like button borders and input fields) must have a contrast ratio of at least **3.0:1**.

---

## 2. Automating Contrast with Axe-Core

The easiest way to validate contrast is to use the Axe-Core integration we built in the previous tutorials. The `color-contrast` rule is built directly into Axe.

When Axe scans the page, it uses JavaScript to compute the computed CSS color of every text element and mathematically compares it against the DOM element rendering directly behind it.

```java
import com.deque.html.axecore.results.Results;
import com.deque.html.axecore.selenium.AxeBuilder;
@Test
public void testColorContrastWithAxe() {
    driver.get("https://practice.mycodeyatra.com/themes");
    // Configure Axe to ONLY run the color-contrast rule
    AxeBuilder builder = new AxeBuilder().withOnlyRules("color-contrast");
    Results axeResults = builder.analyze(driver);
    Assert.assertEquals(axeResults.getViolations().size(), 0, 
        "CRITICAL: Found text elements that violate the 4.5:1 WCAG contrast ratio!");
}
```

---

## 3. The Limitations of Axe Contrast Scanning

Axe is incredibly smart, but it is not magic. It relies on the CSS Object Model (CSSOM). 

If you have a `<div>` with white text, and that `<div>` is positioned over a dark photograph (using CSS `background-image`), Axe-Core **cannot** determine the color of the photograph. Axe will throw an "Incomplete" warning, stating that a human must manually verify the contrast against the image.

To solve this mathematically using pure Selenium, we can extract the CSS colors natively!

---

## 4. Native Selenium CSS Validation

If you want to explicitly validate the computed color of a specific button without running a full Axe scan, you can use Selenium's native `getCssValue()` method.

When Selenium retrieves a color from CSS, it always returns it in `rgba(R, G, B, A)` format.

```java
import org.openqa.selenium.By;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.support.Color;
@Test
public void testPrimaryButtonContrastNatively() {
    driver.get("https://practice.mycodeyatra.com/");
    WebElement primaryBtn = driver.findElement(By.id("checkout-btn"));
    // 1. Get the computed CSS colors
    String textColorRgba = primaryBtn.getCssValue("color");
    String backgroundColorRgba = primaryBtn.getCssValue("background-color");
    // 2. Use Selenium's Color class to convert RGBA to HEX
    String textHex = Color.fromString(textColorRgba).asHex();
    String bgHex = Color.fromString(backgroundColorRgba).asHex();
    System.out.println("Text Color: " + textHex); // e.g., #ffffff
    System.out.println("Background Color: " + bgHex); // e.g., #0056b3
    // 3. (Optional) You can pass these Hex values to a Java Contrast Calculator library 
    // to manually assert the 4.5:1 ratio!
}
```

---

## 5. Theme Toggling and Dark Mode

Modern web applications often have "Dark Mode" and "Light Mode" toggles. 
A common accessibility bug is when a developer designs perfect contrast for Light Mode, but forgets to update the text color in Dark Mode (resulting in dark gray text on a black background).

You must test your contrast in **both** states!

```java
@Test
public void testContrastInDarkAndLightMode() {
    driver.get("https://practice.mycodeyatra.com/dashboard");
    // 1. Test Light Mode Contrast
    AxeBuilder builder = new AxeBuilder().withOnlyRules("color-contrast");
    Assert.assertEquals(builder.analyze(driver).getViolations().size(), 0, 
        "Contrast failed in Light Mode!");
    // 2. Toggle Dark Mode
    driver.findElement(By.id("dark-mode-toggle")).click();
    // 3. Test Dark Mode Contrast!
    Assert.assertEquals(builder.analyze(driver).getViolations().size(), 0, 
        "Contrast failed in Dark Mode!");
}
```

## Conclusion

Color contrast is one of the most frequently violated WCAG rules, but it is also one of the easiest to automate. By combining Axe-Core's `color-contrast` rule with native Selenium `getCssValue()` assertions across all application themes, you guarantee a readable experience for every user.

In our final tutorial of the Accessibility Testing series, we will dive into the incredibly critical world of **Keyboard Navigation**!
