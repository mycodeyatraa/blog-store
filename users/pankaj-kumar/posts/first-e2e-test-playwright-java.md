---
title: First E2E Test - Playwright Java Foundations
date: 09-Jan-2026
lastUpdated: 09-Jan-2026
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
  Build a complete end-to-end user registration and submission flow targeting live components at practice.mycodeyatra.com.
readTime: 10 min read
---

# First E2E Test - Playwright Java Foundations

Building a complete end-to-end (E2E) test requires bringing together page objects, user interactions, web-first assertions, and clean JUnit 5 test fixtures.

In this tutorial, we will automate a real-world user registration workflow targeting **https://practice.mycodeyatra.com/form-practice**.

---

## 1. Page Object Design (`src/main/java/com/mycodeyatra/pages/FirstE2EPage.java`)

```java
package com.mycodeyatra.pages;
 
import com.microsoft.playwright.Page;
import com.microsoft.playwright.Locator;
 
public class FirstE2EPage {
    private final Page page;
    private final Locator usernameInput;
    private final Locator emailInput;
    private final Locator submitBtn;
    private final Locator successBanner;
 
    public FirstE2EPage(Page page) {
        this.page = page;
        this.usernameInput = page.locator("#username");
        this.emailInput = page.locator("#email");
        this.submitBtn = page.locator("#submit-btn");
        this.successBanner = page.locator(".success-banner");
    }
 
    public void navigateToForm() {
        page.navigate("https://practice.mycodeyatra.com/form-practice");
    }
 
    public void fillAndSubmitForm(String username, String email) {
        usernameInput.fill(username);
        emailInput.fill(email);
        submitBtn.click();
    }
 
    public Locator getSuccessBanner() {
        return successBanner;
    }
}
```

---

## 2. Executable Test Suite (`src/test/java/com/mycodeyatra/tests/FirstE2ETest.java`)

```java
package com.mycodeyatra.tests;
 
import com.microsoft.playwright.*;
import com.mycodeyatra.pages.FirstE2EPage;
import org.junit.jupiter.api.*;
import static com.microsoft.playwright.assertions.PlaywrightAssertions.assertThat;
 
@DisplayName("End-to-End User Submission Test")
public class FirstE2ETest {
    private static Playwright playwright;
    private static Browser browser;
    private BrowserContext context;
    private Page page;
 
    @BeforeAll
    static void launch() {
        playwright = Playwright.create();
        browser = playwright.chromium().launch(new BrowserType.LaunchOptions().setHeadless(true));
    }
 
    @BeforeEach
    void setup() {
        context = browser.newContext();
        page = context.newPage();
    }
 
    @Test
    @DisplayName("Verify Successful Form Submission")
    void testFormSubmissionE2E() {
        FirstE2EPage e2ePage = new FirstE2EPage(page);
        e2ePage.navigateToForm();
        e2ePage.fillAndSubmitForm("Pankaj Kumar", "pankaj@mycodeyatra.com");
 
        assertThat(e2ePage.getSuccessBanner()).isVisible();
    }
 
    @AfterEach
    void teardown() {
        context.close();
    }
 
    @AfterAll
    static void closeAll() {
        browser.close();
        playwright.close();
    }
}
```

---

## 3. Key Takeaways

1. **Keep Tests Focused**: Each `@Test` method should validate a single business flow.
2. **Encapsulate Locators**: Never expose `Locator` definitions directly in your test classes.

