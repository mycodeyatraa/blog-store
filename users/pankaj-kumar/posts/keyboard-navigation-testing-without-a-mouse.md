---
title: Keyboard Navigation: Testing Without a Mouse
date: 25-Oct-2024
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [selenium, java, accessibility, keyboard-navigation, actions, ui-automation]
category: UI Automation
categories: [UI Automation, Selenium, Java]
excerpt: >-
  Abandon the mouse. Learn how to simulate users with motor disabilities by exclusively using Selenium's Keyboard Actions to validate focus management and tab-indexing.
readTime: 5 min read
---

# Keyboard Navigation: Testing Without a Mouse

In our previous articles, we used Axe-Core to scan our HTML for WCAG violations. While this catches structural issues like missing ARIA attributes and poor color contrast, it cannot test whether your application is actually *operable*.

According to the WCAG **Operable** principle, every single interactive element on a web page must be reachable and usable using *only* a keyboard. 

If a dropdown menu only opens when you hover over it with a mouse, your site is completely broken for users with motor disabilities who rely on the `Tab` key or specialized sip-and-puff switches to navigate.

In this article, we will write Selenium tests that strictly forbid the use of the mouse, validating focus management and keyboard operability.

---

## 1. The Rules of Keyboard Navigation

Before we automate, we must understand how a keyboard user navigates the web:
- **`Tab`**: Moves focus to the *next* interactive element (Link, Button, Input).
- **`Shift + Tab`**: Moves focus to the *previous* interactive element.
- **`Enter`**: Activates a Link or submits a Form.
- **`Spacebar`**: Clicks a Button or toggles a Checkbox.
- **`Escape`**: Closes modals, dropdowns, and popup menus.

Our goal is to simulate this exact sequence using Selenium's `Actions` and `Keys` APIs.

---

## 2. Validating Focus Management

When a user presses `Tab`, the browser applies a "Focus" state to the active element. In Selenium, we can locate this active element at any time using `driver.switchTo().activeElement()`.

Let's write a test that verifies a user can tab through a login form, verifying that the focus lands on the correct elements in the correct sequential order (known as the `tabindex` order).

```java
package com.mycodeyatra.accessibility;
 
import org.openqa.selenium.By;
import org.openqa.selenium.Keys;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.interactions.Actions;
import org.testng.Assert;
import org.testng.annotations.AfterMethod;
import org.testng.annotations.BeforeMethod;
import org.testng.annotations.Test;
 
public class KeyboardNavigationTest {
 
    private WebDriver driver;
 
    @BeforeMethod
    public void setup() {
        driver = new ChromeDriver();
    }
 
    @Test
    public void verifyLoginFormKeyboardOperability() {
        driver.get("https://mycodeyatra.com/login");
 
        Actions actions = new Actions(driver);
 
        // 1. Find the first element on the page (usually a "Skip to Content" link or the logo)
        WebElement logo = driver.findElement(By.id("navbar-logo"));
        logo.click(); // Click once just to set the initial focus on the page
 
        // 2. Press TAB to move to the Username field
        actions.sendKeys(Keys.TAB).perform();
        WebElement activeElement = driver.switchTo().activeElement();
        Assert.assertEquals(activeElement.getAttribute("id"), "username", "Focus did not land on Username field!");
 
        // 3. Type the username without calling username.sendKeys()
        actions.sendKeys("adminUser").perform();
 
        // 4. Press TAB to move to the Password field
        actions.sendKeys(Keys.TAB).perform();
        activeElement = driver.switchTo().activeElement();
        Assert.assertEquals(activeElement.getAttribute("id"), "password", "Focus did not land on Password field!");
 
        // 5. Type password
        actions.sendKeys("SecureP@ss123").perform();
 
        // 6. Press TAB to move to the "Remember Me" Checkbox
        actions.sendKeys(Keys.TAB).perform();
        activeElement = driver.switchTo().activeElement();
        Assert.assertEquals(activeElement.getAttribute("type"), "checkbox", "Focus did not land on Checkbox!");
 
        // 7. Toggle checkbox using the SPACEBAR
        actions.sendKeys(Keys.SPACE).perform();
        Assert.assertTrue(activeElement.isSelected(), "Spacebar failed to check the box!");
 
        // 8. Press TAB to the Login Button and hit ENTER
        actions.sendKeys(Keys.TAB).perform();
        actions.sendKeys(Keys.ENTER).perform();
 
        // 9. Verify the login was successful
        Assert.assertTrue(driver.getCurrentUrl().contains("dashboard"), "Keyboard Login failed!");
    }
 
    @AfterMethod
    public void teardown() {
        driver.quit();
    }
}
```

---

## 3. The "Skip to Main Content" Link

One of the most annoying experiences for a keyboard user is having to press `Tab` 50 times to get past the massive top-navigation bar on every single page load.

WCAG requires a "Bypass Block" mechanism. In modern web design, this is implemented as a **"Skip to Main Content"** link. This link is usually visually hidden, but appears the moment a user presses `Tab` after the page loads. Pressing `Enter` on it instantly shifts the focus past the navigation bar and directly into the main `<main>` HTML tag.

You can automate this verification:

```java
@Test
public void verifySkipToContentLink() {
    driver.get("https://mycodeyatra.com");
 
    Actions actions = new Actions(driver);
 
    // Press TAB immediately after page load
    actions.sendKeys(Keys.TAB).perform();
 
    WebElement active = driver.switchTo().activeElement();
 
    // The active element MUST be the skip link, and it MUST become visible
    Assert.assertEquals(active.getText(), "Skip to main content");
    Assert.assertTrue(active.isDisplayed(), "Skip link remained hidden when focused!");
 
    // Press ENTER to skip the nav
    actions.sendKeys(Keys.ENTER).perform();
 
    // Verify focus shifted to the main content area
    WebElement nextActive = driver.switchTo().activeElement();
    Assert.assertEquals(nextActive.getTagName(), "main", "Did not skip to main content!");
}
```

---

## System Architecture

Here is the data flow of a Keyboard Navigation Test:

![diagram_1](https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/posts/keyboard-navigation-testing-without-a-mouse/images/diagram_1.png)

## Conclusion

By writing tests that completely ban the use of `WebElement.click()`, we force ourselves to experience the application exactly as a motor-impaired user would. We validate that the logical flow of the DOM (`tabindex`) makes sense, and that custom Javascript widgets correctly listen for `Space` and `Enter` keydown events.

In our final article of Phase 9, we will dive into **Screen Reader Attributes**! We will learn how to parse and validate ARIA (Accessible Rich Internet Applications) tags to ensure visually impaired users hear exactly what they are supposed to.
