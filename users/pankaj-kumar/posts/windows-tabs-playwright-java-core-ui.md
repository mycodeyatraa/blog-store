---
title: Windows & Tabs - Playwright Java Core UI
date: 17-Jan-2026
lastUpdated: 17-Jan-2026
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
  Master Windows & Tabs in Playwright Java! Learn production-grade implementation with hands-on practice.mycodeyatra.com tutorials.
readTime: 9 min read
---

# Windows & Tabs in Playwright Java Core UI

In enterprise UI test automation, handling complex web components like **Windows & Tabs** requires robust wait strategies and native API support. This tutorial covers **Windows & Tabs** using Playwright Java targeting live components at **https://practice.mycodeyatra.com**.

---

## 1. Architectural Overview & Component Focus

```
 +---------------------------------------------------+
 |  JUnit 5 Test Suite (@Test / PlaywrightAssertions) |
 +---------------------------------------------------+
                          |
                          v
 +---------------------------------------------------+
 |  Playwright Java Page Objects (src/main/java)      |
 +---------------------------------------------------+
                          |
                          v
 +---------------------------------------------------+
 |  Practice App (https://practice.mycodeyatra.com)  |
 +---------------------------------------------------+
```

- **Repository Path**: Source code for this module is checked into `Repository/mcyt-plw-java`.
- **Automatic Piercing**: Playwright's CSS engine automatically pierces Shadow DOM boundaries without extra JS injection.
- **Auto-Waiting**: Automatically waits for elements to be visible, enabled, and stable before interacting.

---

## 2. Production Page Object Implementation (`src/main/java/com/mycodeyatra/pages/WebTablesPage.java`)

```java
package com.mycodeyatra.pages;
 
import com.microsoft.playwright.Page;
import com.microsoft.playwright.Locator;
 
public class WebTablesPage {
    private final Page page;
    private final Locator tableRows;
 
    public WebTablesPage(Page page) {
        this.page = page;
        this.tableRows = page.locator("table tr");
    }
 
    public void navigateToTablesPage() {
        page.navigate("https://practice.mycodeyatra.com/tables");
    }
 
    public Locator getRowByText(String text) {
        return page.locator("table tr:has-text('" + text + "')");
    }
 
    public int getRowCount() {
        return tableRows.count();
    }
}
```

---

## 3. Executable Test Suite (`src/test/java/com/mycodeyatra/tests/PlaywrightCoreUITest.java`)

```java
package com.mycodeyatra.tests;
 
import com.microsoft.playwright.*;
import com.mycodeyatra.pages.WebTablesPage;
import org.junit.jupiter.api.*;
import static com.microsoft.playwright.assertions.PlaywrightAssertions.assertThat;
 
public class PlaywrightCoreUITest {
    private static Playwright playwright;
    private static Browser browser;
    private BrowserContext context;
    private Page page;
 
    @BeforeAll
    static void launchBrowser() {
        playwright = Playwright.create();
        browser = playwright.chromium().launch(new BrowserType.LaunchOptions().setHeadless(true));
    }
 
    @BeforeEach
    void createContext() {
        context = browser.newContext();
        page = context.newPage();
    }
 
    @Test
    @DisplayName("Validate Windows & Tabs on practice.mycodeyatra.com")
    void testCoreUIWorkflow() {
        WebTablesPage tablesPage = new WebTablesPage(page);
        tablesPage.navigateToTablesPage();
        
        assertThat(tablesPage.getRowByText("Admin")).isVisible();
    }
 
    @AfterEach
    void closeContext() {
        context.close();
    }
 
    @AfterAll
    static void closeBrowser() {
        browser.close();
        playwright.close();
    }
}
```

---

## 4. Key Takeaways & Best Practices

1. **Native Shadow DOM Support**: Use simple Playwright locators to query elements inside open Shadow Roots.
2. **Event Listening**: Intercept dialogs and download events using lambda listeners before clicking trigger buttons.
3. **Target Environment**: Run UI regression suites against `https://practice.mycodeyatra.com`.
