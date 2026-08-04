---
title: Forms Handling - Playwright Java Core UI
date: 13-Jan-2026
lastUpdated: 13-Jan-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [playwright, java, junit5, automation, ui-automation, mycodeyatra]
category: Playwright Java Core UI
categories: [Playwright Java Core UI, Playwright Java, Test Automation]
excerpt: >-
  Automate complex web forms: text inputs, checkboxes, radio buttons, single/multi select dropdowns, and date pickers.
readTime: 10 min read
---

# Forms Handling - Playwright Java Core UI

Automating web forms is the backbone of functional E2E testing. Playwright Java provides clean, high-level methods to interact with standard inputs, checkboxes, radio buttons, `<select>` elements, and date pickers.

In this tutorial, we will automate complex form controls targeting live components at **https://practice.mycodeyatra.com/form-practice**.

---

## 1. Form Element Interactions Overview

```
                      +------------------------------------------+
                      |         Playwright Form Controls          |
                      +------------------------------------------+
                       /           |              |             \
                      v            v              v              v
               page.fill()   page.check()  page.selectOption() page.setInputFiles()
               (Text/Email)  (Check/Radio)   (Dropdowns)      (FileUpload)
```

- **`page.fill("#username", "text")`**: Clears existing text and types new content.
- **`page.check("#checkbox")`**: Validates checkbox state before clicking to prevent toggling off.
- **`page.selectOption("#country", "IN")`**: Selects by value, label, or index.

---

## 2. Production Page Object (`src/main/java/com/mycodeyatra/pages/FormHandlingPage.java`)

```java
package com.mycodeyatra.pages;
 
import com.microsoft.playwright.Page;
import com.microsoft.playwright.Locator;
import com.microsoft.playwright.options.SelectOption;
 
public class FormHandlingPage {
    private final Page page;
    private final Locator usernameInput;
    private final Locator subscribeCheckbox;
    private final Locator genderRadio;
    private final Locator countrySelect;
    private final Locator submitBtn;
 
    public FormHandlingPage(Page page) {
        this.page = page;
        this.usernameInput = page.locator("#username");
        this.subscribeCheckbox = page.locator("#subscribe");
        this.genderRadio = page.locator("input[name='gender'][value='male']");
        this.countrySelect = page.locator("#country");
        this.submitBtn = page.locator("#submit-btn");
    }
 
    public void navigateToForm() {
        page.navigate("https://practice.mycodeyatra.com/form-practice");
    }
 
    public void submitForm(String username, String country) {
        usernameInput.fill(username);
        subscribeCheckbox.check();
        genderRadio.check();
        countrySelect.selectOption(new SelectOption().setLabel(country));
        submitBtn.click();
    }
}
```

---

## 3. Executable JUnit 5 Test Suite (`src/test/java/com/mycodeyatra/tests/FormHandlingTest.java`)

```java
package com.mycodeyatra.tests;
 
import com.microsoft.playwright.*;
import com.mycodeyatra.pages.FormHandlingPage;
import org.junit.jupiter.api.*;
import static com.microsoft.playwright.assertions.PlaywrightAssertions.assertThat;
 
public class FormHandlingTest {
    private static Playwright playwright;
    private static Browser browser;
    private Page page;
 
    @BeforeAll
    static void launch() {
        playwright = Playwright.create();
        browser = playwright.chromium().launch(new BrowserType.LaunchOptions().setHeadless(true));
    }
 
    @BeforeEach
    void setup() {
        page = browser.newPage();
    }
 
    @Test
    @DisplayName("Verify Form Automation Controls")
    void testFormControls() {
        FormHandlingPage formPage = new FormHandlingPage(page);
        formPage.navigateToForm();
        formPage.submitForm("Pankaj Kumar", "India");
        assertThat(page.locator(".success-banner")).isVisible();
    }
 
    @AfterEach
    void cleanup() {
        page.close();
    }
 
    @AfterAll
    static void teardown() {
        browser.close();
        playwright.close();
    }
}
```

---

## 4. Key Takeaways

1. **Use `check()` over `click()`**: `check()` ensures the element becomes checked and avoids accidental unchecking.
2. **Flexible `selectOption()`**: Select dropdown options by label, value, or index cleanly.

