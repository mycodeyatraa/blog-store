---
title: Locators Deep Dive - Playwright Java Foundations
date: 07-Jan-2026
lastUpdated: 07-Jan-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [playwright, java, junit5, automation, testing, mycodeyatra]
category: Playwright Java Foundations
categories: [Playwright Java Foundations, Playwright Java, Test Automation]
excerpt: >-
  Master Playwright's user-facing locators: getByRole, getByText, getByLabel, getByTestId, and CSS chaining for resilient tests.
readTime: 10 min read
---

# Locators Deep Dive - Playwright Java Foundations

Locators are the core building blocks of Playwright Java automation. Unlike Selenium `WebElement` objects which represent a static snapshot of an element at a point in time, a Playwright `Locator` represents an **auto-waiting element query**.

---

## 1. Why User-Facing Locators Matter

Traditional automation tests frequently rely on brittle implementation details like complex XPath expressions (`//div[4]/span[2]/input`) or auto-generated CSS classes (`.css-192jx8`). When developers refactor class names or DOM structures, tests break immediately.

Playwright advocates for **user-facing locators** that reflect how real users interact with the webpage:

```
USER-FACING LOCATOR PHILOSOPHY:
"Find the button named 'Submit'" -> page.getByRole(AriaRole.BUTTON, new Page.GetByRoleOptions().setName("Submit"))
"Find the input labeled 'Email'"  -> page.getByLabel("Email Address")
```

---

## 2. Priority Order of Locators

1. **`page.getByRole(AriaRole, options)`**: Locates elements by ARIA role (`BUTTON`, `CHECKBOX`, `HEADING`, `DIALOG`).
2. **`page.getByLabel(text)`**: Locates form inputs by associated `<label>` text.
3. **`page.getByPlaceholder(text)`**: Locates input elements by placeholder text.
4. **`page.getByText(text)`**: Locates non-interactive elements by visible text content.
5. **`page.getByTestId(id)`**: Locates elements by explicit testing attributes (`data-testid`).

---

## 3. Production Page Object (`src/main/java/com/mycodeyatra/pages/LocatorsPage.java`)

```java
package com.mycodeyatra.pages;
 
import com.microsoft.playwright.Page;
import com.microsoft.playwright.Locator;
import com.microsoft.playwright.options.AriaRole;
 
public class LocatorsPage {
    private final Page page;
    private final Locator fullNameInput;
    private final Locator submitBtn;
    private final Locator successBanner;
 
    public LocatorsPage(Page page) {
        this.page = page;
        this.fullNameInput = page.getByLabel("Full Name");
        this.submitBtn = page.getByRole(AriaRole.BUTTON, new Page.GetByRoleOptions().setName("Submit Form"));
        this.successBanner = page.locator(".success-banner");
    }
 
    public void navigateToForm() {
        page.navigate("https://practice.mycodeyatra.com/form-practice");
    }
 
    public void submitForm(String name) {
        fullNameInput.fill(name);
        submitBtn.click();
    }
 
    public Locator getSuccessBanner() {
        return successBanner;
    }
}
```

---

## 4. Advanced Locator Chaining & Filtering

Playwright allows clean chaining and filtering without complex XPath syntax:

```java
// Filter table rows by text content
Locator row = page.locator("table tr").filter(new Locator.FilterOptions().setHasText("Admin"));
 
// Select child button within a specific container
Locator saveBtn = page.locator(".card-footer").getByRole(AriaRole.BUTTON, new Page.GetByRoleOptions().setName("Save"));
```

