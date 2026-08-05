---
title: "Managing Lifecycle Hooks in Playwright Java Cucumber Suites"
date: "14-Aug-2026"
lastUpdated: "14-Aug-2026"
author: "pankaj-kumar"
authorName: "Pankaj Kumar"
authorRole: "Automation Architect"
authorAvatar: "https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG"
authorBio: "Automation Architect"
authorGithub: "https://github.com/pankajhyd"
authorLinkedin: "https://www.linkedin.com/in/pankaj-kumar-94a2b227/"
tags: ["Playwright", "Java", "Cucumber", "Hooks", "Lifecycle"]
category: "Automation Testing"
categories: ["Automation Testing", "Playwright Java", "Hooks"]
excerpt: "Configure @Before, @After, @BeforeAll, and @AfterAll hooks to manage Playwright BrowserContext, page initialization, trace recording, and failure screenshots."
readTime: "7 min read"
---

# Managing Lifecycle Hooks in Playwright Java Cucumber Suites

Hooks in Cucumber-JVM execute before or after scenarios to set up browser contexts, isolate state, capture failure evidence, and release system resources.

---

## 1. Centralized Hooks Class

```java
// src/test/java/com/mycodeyatra/steps/Hooks.java
package com.mycodeyatra.steps;
 
import com.microsoft.playwright.*;
import io.cucumber.java.*;
import java.nio.file.Paths;
 
public class Hooks {
    private static Playwright playwright;
    private static Browser browser;
    private BrowserContext context;
    private Page page;
 
    @BeforeAll
    public static void globalSetup() {
        playwright = Playwright.create();
        browser = playwright.chromium().launch(new BrowserType.LaunchOptions().setHeadless(true));
    }
 
    @Before
    public void setup(Scenario scenario) {
        context = browser.newContext(new Browser.NewContextOptions().setViewportSize(1920, 1080));
        context.tracing().start(new Tracing.StartOptions()
                .setScreenshots(true)
                .setSnapshots(true)
                .setSources(true));
        page = context.newPage();
    }
 
    @After
    public void teardown(Scenario scenario) {
        if (scenario.isFailed()) {
            byte[] screenshot = page.screenshot(new Page.ScreenshotOptions().setFullPage(true));
            scenario.attach(screenshot, "image/png", scenario.getName() + "_failure");
 
            context.tracing().stop(new Tracing.StopOptions()
                    .setPath(Paths.get("target/traces/" + scenario.getName().replaceAll("\\s+", "_") + ".zip")));
        } else {
            context.tracing().stop(new Tracing.StopOptions());
        }
        context.close();
    }
 
    @AfterAll
    public static void globalTeardown() {
        if (browser != null) browser.close();
        if (playwright != null) playwright.close();
    }
 
    public Page getPage() {
        return page;
    }
}
```

---

## 2. Tagged Hooks Execution

You can target hooks to specific scenarios using tags:

```java
// Conditional hooks for API or Database setup
@Before("@RequiresDB")
public void seedTestData() {
    System.out.println("Seeding database records for test...");
}
 
@After("@CleanCache")
public void clearCookiesAndLocalStorage() {
    page.context().clearCookies();
}
```

---

## 3. Execution Lifecycle Order

```
@BeforeAll (Once per test suite run)
   │
   ├── @Before (Before Scenario 1)
   │     ├── Scenario 1 Steps Execution
   │     └── @After (Failure Screenshot / Trace Capture)
   │
   ├── @Before (Before Scenario 2)
   │     ├── Scenario 2 Steps Execution
   │     └── @After (Teardown Context)
   │
@AfterAll (Once per test suite shutdown)
```
