---
title: Step Definitions, Glue Code & Hooks - Playwright Java Design Patterns
date: 25-Feb-2026
lastUpdated: 25-Feb-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [playwright, java, junit5, automation, design-patterns, mycodeyatra]
category: Playwright Java Design Patterns
categories: [Playwright Java Design Patterns, Playwright Java, Test Automation]
excerpt: >-
  Implement Cucumber step definition glue code and manage test lifecycle fixtures with @Before and @After hooks in Playwright Java.
readTime: 9 min read
---

# Step Definitions, Glue Code & Hooks - Playwright Java Design Patterns

Step Definitions act as the glue code mapping Gherkin steps to Playwright Java Page Object methods and browser automation actions.

---

## 1. Step Definition Glue Code Implementation

```java
package com.mycodeyatra.steps;
 
import com.microsoft.playwright.*;
import io.cucumber.java.en.*;
import static com.microsoft.playwright.assertions.PlaywrightAssertions.assertThat;
 
public class LoginStepDefinitions {
    private Page page;
 
    @Given("the user navigates to the practice login portal")
    public void navigateToLogin() {
        page.navigate("https://practice.mycodeyatra.com/login");
    }
 
    @When("the user enters valid username {string} and password {string}")
    public void enterCredentials(String username, String password) {
        page.fill("#username", username);
        page.fill("#password", password);
    }
}
```

---

## 2. Cucumber Lifecycle Hooks

```java
@Before
public void setupBrowserSession() {
    playwright = Playwright.create();
    browser = playwright.chromium().launch();
    context = browser.newContext();
    page = context.newPage();
}
 
@After
public void teardownBrowserSession(Scenario scenario) {
    if (scenario.isFailed()) {
        byte[] screenshot = page.screenshot();
        scenario.attach(screenshot, "image/png", "Failure Screenshot");
    }
    context.close();
    browser.close();
}
```

---

## 3. Key Takeaways

1. **Clean Decoupling**: Keep step definitions thin by delegating DOM actions to Page Objects.
2. **Failure Screenshots**: Automatically attach screenshots to Cucumber reports upon scenario failure.

