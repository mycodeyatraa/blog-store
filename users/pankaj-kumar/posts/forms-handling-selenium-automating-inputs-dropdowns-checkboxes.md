---
title: Forms Handling in Selenium: Automating Inputs, Dropdowns, and Checkboxes
date: 12-Jul-2024
lastUpdated: 11-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [selenium, java, forms, inputs, dropdowns, select-class]
category: UI Automation
categories: [UI Automation, Selenium, Java]
excerpt: >-
  A complete guide to automating forms handling in Selenium WebDriver. Learn how to interact with inputs, radio buttons, checkboxes, single-select and multi-select dropdowns in Java.
readTime: 7 min read
---

# Forms Handling in Selenium: Automating Inputs, Dropdowns, and Checkboxes

> 📅 **Last Updated:** 11-Jun-2026

Forms are the primary interaction layer of most business web applications. Whether it is a registration screen, checkout portal, or search hub, automation scripts must handle inputs, radio selections, checkboxes, and multi-select dropdowns reliably.

In this tutorial, we will explore how to interact with standard form components and automate complex multi-select validations using Selenium WebDriver and the `Select` support class in Java.

---

## 🧭 Form Interaction Decision Engine

Different form elements require different Selenium interaction methods. The flowchart below maps the decision process for selecting the correct interaction strategy based on DOM element types:

![diagram_1](https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/posts/forms-handling-selenium-automating-inputs-dropdowns-checkboxes/images/diagram_1.png)

---

## 🛠️ Step-by-Step Code Walkthrough

Here is the complete TestNG implementation demonstrating form automation. It navigates to a multi-field validation form, fills text fields, selects radio and checkbox parameters, handles both single and multi-select dropdown panels, and validates the output summaries.

```java
package com.mycodeyatra.tests;
import io.github.bonigarcia.wdm.WebDriverManager;
import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.chrome.ChromeOptions;
import org.openqa.selenium.support.ui.ExpectedConditions;
import org.openqa.selenium.support.ui.Select;
import org.openqa.selenium.support.ui.WebDriverWait;
import org.testng.Assert;
import org.testng.annotations.AfterMethod;
import org.testng.annotations.BeforeMethod;
import org.testng.annotations.Test;
import java.time.Duration;
public class FormTest {
    private WebDriver driver;
    private WebDriverWait wait;
    @BeforeMethod
    public void setUp() {
        WebDriverManager.chromedriver().setup();
        ChromeOptions options = new ChromeOptions();
        options.addArguments("--remote-allow-origins=*");
        options.addArguments("--headless=new");
        options.addArguments("--window-size=1920,1080");
        driver = new ChromeDriver(options);
        driver.manage().window().maximize();
        driver.manage().timeouts().implicitlyWait(Duration.ofSeconds(10));
        wait = new WebDriverWait(driver, Duration.ofSeconds(5));
    }
    @Test
    public void testFormSubmissionFlow() {
        System.out.println("Navigating to: https://practice.mycodeyatra.com/");
        driver.get("https://practice.mycodeyatra.com/");
        System.out.println("Opening Sandbox Arena...");
        WebElement sandboxLink = wait.until(ExpectedConditions.elementToBeClickable(By.xpath("//span[contains(text(),'Sandbox Arena')]")));
        sandboxLink.click();
        System.out.println("Opening Form Practice Page...");
        WebElement formTile = wait.until(ExpectedConditions.elementToBeClickable(By.xpath("//h3[text()='Form Practice']")));
        formTile.click();
        // 1. Text Inputs (Name, Email, Phone)
        System.out.println("Filling text inputs...");
        driver.findElement(By.xpath("//input[@data-testid='full-name']")).sendKeys("Jane Doe");
        driver.findElement(By.xpath("//input[@data-testid='email']")).sendKeys("jane.doe@example.com");
        driver.findElement(By.xpath("//input[@data-testid='phone']")).sendKeys("1234567890");
        // 2. Radio Buttons (Gender - Female)
        System.out.println("Selecting radio button...");
        driver.findElement(By.xpath("//input[@data-testid='gender-female']")).click();
        // 3. Checkboxes (Interests - Automation & Testing)
        System.out.println("Selecting interest checkboxes...");
        driver.findElement(By.xpath("//input[@data-testid='interest-automation']")).click();
        driver.findElement(By.xpath("//input[@data-testid='interest-testing']")).click();
        // 4. Select Single Dropdown (Country - India)
        System.out.println("Selecting country from single select dropdown...");
        WebElement countryDropdown = driver.findElement(By.xpath("//select[@data-testid='country-select']"));
        Select countrySelect = new Select(countryDropdown);
        countrySelect.selectByVisibleText("India");
        // 5. Select Multi-Dropdown (Automation Tools - Selenium & Playwright)
        System.out.println("Selecting tools from multi-select dropdown...");
        WebElement toolsDropdown = driver.findElement(By.xpath("//select[@data-testid='tools-multi-select']"));
        Select toolsSelect = new Select(toolsDropdown);
        toolsSelect.selectByValue("Selenium");
        toolsSelect.selectByValue("Playwright");
        // 6. Textarea (Bio)
        System.out.println("Writing bio...");
        driver.findElement(By.xpath("//textarea[@data-testid='bio']")).sendKeys("Senior QA Engineer specializing in automated frameworks.");
        // 7. Submit Form
        System.out.println("Submitting form...");
        driver.findElement(By.xpath("//button[@data-testid='submit-btn']")).click();
        // --- VALIDATIONS ---
        System.out.println("Validating success alert message...");
        WebElement successMsg = wait.until(
            ExpectedConditions.visibilityOfElementLocated(By.xpath("//div[@data-testid='success-msg']"))
        );
        Assert.assertEquals(successMsg.getText(), "Form submitted successfully!", "Form failed submission validation.");
        System.out.println("Validating results summary values...");
        Assert.assertEquals(driver.findElement(By.xpath("//span[@data-testid='result-name']")).getText(), "Jane Doe");
        Assert.assertEquals(driver.findElement(By.xpath("//span[@data-testid='result-email']")).getText(), "jane.doe@example.com");
        Assert.assertEquals(driver.findElement(By.xpath("//span[@data-testid='result-gender']")).getText(), "Female");
        Assert.assertEquals(driver.findElement(By.xpath("//span[@data-testid='result-country']")).getText(), "India");
        Assert.assertTrue(driver.findElement(By.xpath("//span[@data-testid='result-tools']")).getText().contains("Selenium"));
        Assert.assertTrue(driver.findElement(By.xpath("//span[@data-testid='result-tools']")).getText().contains("Playwright"));
        System.out.println("Form handling test validations complete!");
    }
    @AfterMethod
    public void tearDown() {
        if (driver != null) {
            System.out.println("Quitting driver session...");
            driver.quit();
        }
    }
}
```

---

## 💻 Test Execution Logs

Here are the console execution logs from running this TestNG suite using Maven:

```bash
[INFO] Running com.mycodeyatra.tests.FormTest
Navigating to: https://practice.mycodeyatra.com/
Opening Sandbox Arena...
Opening Form Practice Page...
Filling text inputs...
Selecting radio button...
Selecting interest checkboxes...
Selecting country from single select dropdown...
Selecting tools from multi-select dropdown...
Writing bio...
Submitting form...
Validating success alert message...
Validating results summary values...
Form handling test validations complete!
Quitting driver session...
Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 5.926 sec
```

---

## 📊 Dropdown Selection Methods

Selenium's `org.openqa.selenium.support.ui.Select` wrapper provides three primary selection APIs.

| Selection Method | Target Argument | Performance Scope | Best Practices |
| :--- | :--- | :--- | :--- |
| `selectByVisibleText(text)` | Sub-option tag text label | Dynamic rendering safe | Ideal for user-visible localization testing |
| `selectByValue(value)` | HTML `value="..."` attribute | Fast DOM lookup | Ideal for static lists with unique DB IDs |
| `selectByIndex(index)` | Numeric index (0-indexed) | Fast but fragile | Avoid if options shift or load dynamically |

---

## ⚠️ Common Pitfalls

* **Using the wrong element with Select class**: Wrapping an input field, list-item, or custom span wrapper inside `new Select(element)` throws an `UnexpectedTagNameException`. The Select class is strictly reserved for HTML `<select>` elements.
* **Failure to clear inputs before sendKeys**: Invoking `.sendKeys("Jane")` twice on an input container without clearing it between sessions appends the text, yielding `"JaneJane"`. Always invoke `.clear()` beforehand.
* **Hardcoded multi-select option order**: Indices in multi-select boxes change if elements load dynamically or are sorted alphabetically. Prefer `selectByValue` or `selectByVisibleText` for multi-selections.

---

## 🙋 Frequently Asked Questions

> **Q: How do we automate custom dropdown elements (built with divs/spans instead of select tags)?**
>
> **A:** Custom select components (like React-Select or Vuetify panels) cannot use the `Select` class. You must click the list wrapper trigger element to expand the panel, locate the list item element using CSS/XPath, and click the option manually.

> **Q: What is the purpose of deselect methods inside multi-select dropdowns?**
>
> **A:** For `<select multiple>` tags, you can remove individual choices using `.deselectByValue(value)` or wipe out all highlighted choices via `.deselectAll()`. These methods will throw an exception if invoked on a standard single-choice dropdown.
