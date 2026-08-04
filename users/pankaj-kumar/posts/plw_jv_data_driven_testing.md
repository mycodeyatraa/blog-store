---
title: Data Driven Testing - Playwright Java Design Patterns
date: 23-Jan-2026
lastUpdated: 23-Jan-2026
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
  Implement data-driven testing in Playwright Java using JUnit 5 @ParameterizedTest, @CsvSource, @MethodSource, and JSON test data providers.
readTime: 9 min read
---

# Data Driven Testing - Playwright Java Design Patterns

Data-Driven Testing (DDT) is a core design pattern that separates test logic from test data. Rather than hardcoding input parameters inside test methods, DDT allows executing a single test scenario against multiple data sets (such as valid/invalid credentials or localized form inputs).

Playwright Java integrates seamlessly with JUnit 5 Jupiter parameterization features like `@ParameterizedTest`, `@CsvSource`, `@CsvFileSource`, and `@MethodSource`.

---

## 1. Parameterization Architecture in JUnit 5

```
              +-------------------------------------------------+
              |               Test Data Source                  |
              |   (CSV File / JSON File / JUnit MethodSource)   |
              +-------------------------------------------------+
                                       |
                                       v
              +-------------------------------------------------+
              |     JUnit 5 @ParameterizedTest Execution Engine |
              +-------------------------------------------------+
                                       |
                                       v
              +-------------------------------------------------+
              |     Playwright BrowserContext & Page Actions    |
              +-------------------------------------------------+
```

---

## 2. Parameterized Test Examples

```java
// 1. Data-Driven Testing with @CsvSource
@ParameterizedTest
@CsvSource({
    "user1, password123, Valid User",
    "user2, invalidPass, Error User",
    "admin, adminPass, Admin User"
})
void testLoginWithCsvData(String username, String password, String expectedRole) {
    page.navigate("https://practice.mycodeyatra.com/login");
    page.fill("#username", username);
    page.fill("#password", password);
    page.click("#login-btn");
    assertThat(page.locator(".user-role")).containsText(expectedRole);
}
 
// 2. Data-Driven Testing with @MethodSource
@ParameterizedTest
@MethodSource("provideUserData")
void testFormSubmissionWithMethodSource(String fullName, String email) {
    page.navigate("https://practice.mycodeyatra.com/form-practice");
    page.fill("#username", fullName);
    page.fill("#email", email);
    page.click("#submit-btn");
    assertThat(page.locator(".success-banner")).isVisible();
}
 
static Stream<Arguments> provideUserData() {
    return Stream.of(
        Arguments.of("Pankaj Kumar", "pankaj@mycodeyatra.com"),
        Arguments.of("Automation Architect", "architect@mycodeyatra.com")
    );
}
```

---

## 3. Production Page Object (`src/main/java/com/mycodeyatra/pages/design/DataDrivenPage.java`)

```java
package com.mycodeyatra.pages.design;
 
import com.microsoft.playwright.Page;
import com.microsoft.playwright.Locator;
 
public class DataDrivenPage {
    private final Page page;
    private final Locator usernameInput;
    private final Locator emailInput;
    private final Locator submitBtn;
 
    public DataDrivenPage(Page page) {
        this.page = page;
        this.usernameInput = page.locator("#username");
        this.emailInput = page.locator("#email");
        this.submitBtn = page.locator("#submit-btn");
    }
 
    public void navigateToForm() {
        page.navigate("https://practice.mycodeyatra.com/form-practice");
    }
 
    public void submitData(String username, String email) {
        usernameInput.fill(username);
        emailInput.fill(email);
        submitBtn.click();
    }
}
```

---

## 4. Key Takeaways

1. **Use `@CsvFileSource` for Large Data Sets**: Keep test data stored in external `.csv` files under `src/test/resources/data/`.
2. **Combine with Playwright Assertions**: Ensure web-first assertions validate each parameterized iteration independently.

