---
title: Screen Reader Attributes: Validating ARIA Tags
date: 28-Oct-2024
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [selenium, java, accessibility, aria, screen-readers, ui-automation]
category: UI Automation
categories: [UI Automation, Selenium, Java]
excerpt: >-
  Complete the 59-blog curriculum by testing the WCAG 'Perceivable' principle. Learn how to validate ARIA states and live regions to ensure dynamic web apps are perfectly readable by screen readers.
readTime: 5 min read
---

# Screen Reader Attributes: Validating ARIA Tags

Welcome to the **59th and final article** of the Ultimate Selenium Java framework series!

Over the past nine phases, we have built an enterprise-grade framework spanning Page Objects, API mocking, parallel grids, security testing, and visual regression. But we have one final, critical check to make: **Can visually impaired users understand our application?**

When a visually impaired user browses the web, they use software called a "Screen Reader" (like JAWS, NVDA, or Apple VoiceOver) that reads the HTML out loud.

In the early days of the web, this was easy. But today, modern frameworks like React and Angular update the DOM dynamically without reloading the page. If a red error message suddenly appears on the screen, a sighted user sees it instantly. But how does the Screen Reader know it appeared?

The answer is **ARIA** (Accessible Rich Internet Applications) attributes. In this final article, we will write Selenium tests to validate that ARIA state tags update correctly.

---

## 1. What are ARIA Attributes?

ARIA is a set of special HTML attributes that provide semantic meaning to screen readers. They do not change how the application *looks*; they only change how the application *sounds*.

Common ARIA states include:
- `aria-expanded="true/false"`: Used on dropdowns and accordions.
- `aria-hidden="true/false"`: Used to hide decorative elements from the screen reader so they aren't read out loud.
- `aria-invalid="true"`: Used on form fields when the user types invalid data.

Our job as automation engineers is to verify that the frontend developers are toggling these attributes via JavaScript when the user interacts with the page!

---

## 2. Validating State Changes (Accordions)

Let's imagine an FAQ page with an accordion. When you click the Question, the Answer expands downward. 

We must write a test to verify that the `aria-expanded` attribute changes from `false` to `true`, ensuring the screen reader announces "Expanded" to the user.

```java
package com.mycodeyatra.accessibility;
 
import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.chrome.ChromeDriver;
import org.testng.Assert;
import org.testng.annotations.AfterMethod;
import org.testng.annotations.BeforeMethod;
import org.testng.annotations.Test;
 
public class AriaValidationTest {
 
    private WebDriver driver;
 
    @BeforeMethod
    public void setup() {
        driver = new ChromeDriver();
    }
 
    @Test
    public void verifyAccordionAriaState() {
        driver.get("https://mycodeyatra.com/faq");
 
        WebElement accordionButton = driver.findElement(By.id("faq-question-1"));
 
        // 1. Verify initial state is collapsed
        String initialState = accordionButton.getAttribute("aria-expanded");
        Assert.assertEquals(initialState, "false", "Accordion should be collapsed initially!");
 
        // 2. Click the accordion to open it
        accordionButton.click();
 
        // 3. Verify the ARIA state updated to true
        String expandedState = accordionButton.getAttribute("aria-expanded");
        Assert.assertEquals(expandedState, "true", "Screen Reader was not notified of expansion!");
 
        // 4. Verify the hidden content is now visible
        WebElement answerText = driver.findElement(By.id("faq-answer-1"));
        Assert.assertTrue(answerText.isDisplayed(), "Answer text is not visible!");
    }
 
    @AfterMethod
    public void teardown() {
        driver.quit();
    }
}
```

---

## 3. Validating ARIA Live Regions (Toast Notifications)

If you click "Save Profile", and a green "Profile Saved!" toast notification slides onto the screen, the screen reader needs to interrupt whatever it is currently reading and immediately announce the success message.

This is achieved using the `aria-live` attribute.
- `aria-live="polite"`: The screen reader waits until it finishes its current sentence, then reads the update.
- `aria-live="assertive"`: The screen reader immediately interrupts itself to read the update (used for critical errors).

We can validate this implementation:

```java
@Test
public void verifyToastNotificationIsAnnounced() {
    driver.get("https://mycodeyatra.com/profile");
 
    // 1. Click save
    driver.findElement(By.id("save-profile-btn")).click();
 
    // 2. Locate the Toast Notification container
    WebElement toastContainer = driver.findElement(By.id("toast-message"));
 
    // 3. Verify the container has the assertive live region tag
    String liveRegion = toastContainer.getAttribute("aria-live");
    Assert.assertEquals(liveRegion, "assertive", "Screen Reader will ignore the success message!");
 
    // 4. Verify the message text is injected into the container
    Assert.assertEquals(toastContainer.getText(), "Profile Saved Successfully!");
}
```

---

## System Architecture

Here is the data flow for Screen Reader interaction, highlighting why testing the `getAttribute` state is so critical:

![diagram_1](https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/posts/screen-reader-attributes-aria-tags-validation/images/diagram_1.png)

## Curriculum Conclusion

Congratulations! Over the course of **59 articles**, you have evolved from writing basic `driver.get()` scripts to architecting a world-class, enterprise-grade test automation framework. 

You have learned:
- **Phase 1-3:** OOP Design, Page Object Models, and Data-Driven Testing.
- **Phase 4-5:** Selenium Grid, Docker, CI/CD, and Cloud Execution.
- **Phase 6:** Network Mocking and CDP manipulation.
- **Phase 7:** Advanced Security Testing (DAST & JWTs).
- **Phase 8:** AI-Driven Visual Regression.
- **Phase 9:** WCAG Compliance and Accessibility.

You now possess the knowledge to lead automation engineering teams at the highest levels of the industry. 

Thank you for following the MyCodeYatra Selenium Java curriculum. Keep coding, keep testing, and continue building a faster, more secure, and accessible web for everyone!
