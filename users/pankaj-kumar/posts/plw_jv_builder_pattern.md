---
title: Builder Pattern - Playwright Java Design Patterns
date: 26-Jan-2026
lastUpdated: 26-Jan-2026
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
  Construct complex test data payloads using the Builder Pattern in Playwright Java automation.
readTime: 9 min read
---

# Builder Pattern - Playwright Java Design Patterns

The Builder Pattern is a creational design pattern that allows constructing complex objects step by step. When automating web forms with 15+ fields (such as user registration or checkout forms), constructor overloading leads to unmaintainable code.

Using the Builder Pattern, you can create readable, immutable test data objects with sensible default values.

---

## 1. Test Data Builder Pattern Model

```java
// 1. Immutable User Data Model with Builder
public class UserData {
    private final String firstName;
    private final String lastName;
    private final String email;
    private final String country;
 
    private UserData(Builder builder) {
        this.firstName = builder.firstName;
        this.lastName = builder.lastName;
        this.email = builder.email;
        this.country = builder.country;
    }
 
    public String getFirstName() { return firstName; }
    public String getLastName() { return lastName; }
    public String getEmail() { return email; }
    public String getCountry() { return country; }
 
    public static class Builder {
        private String firstName = "DefaultFirst";
        private String lastName = "DefaultLast";
        private String email = "test@mycodeyatra.com";
        private String country = "India";
 
        public Builder firstName(String val) { this.firstName = val; return this; }
        public Builder lastName(String val) { this.lastName = val; return this; }
        public Builder email(String val) { this.email = val; return this; }
        public Builder country(String val) { this.country = val; return this; }
 
        public UserData build() { return new UserData(this); }
    }
}
 
// 2. Test Execution with Builder
UserData user = new UserData.Builder()
    .firstName("Pankaj")
    .email("pankaj@mycodeyatra.com")
    .build();
```

---

## 2. Production Page Object (`src/main/java/com/mycodeyatra/pages/design/UserRegistrationBuilderPage.java`)

```java
package com.mycodeyatra.pages.design;
 
import com.microsoft.playwright.Page;
import com.microsoft.playwright.Locator;
 
public class UserRegistrationBuilderPage {
    private final Page page;
    private final Locator usernameInput;
    private final Locator emailInput;
    private final Locator submitBtn;
 
    public UserRegistrationBuilderPage(Page page) {
        this.page = page;
        this.usernameInput = page.locator("#username");
        this.emailInput = page.locator("#email");
        this.submitBtn = page.locator("#submit-btn");
    }
 
    public void navigateToForm() {
        page.navigate("https://practice.mycodeyatra.com/form-practice");
    }
 
    public void registerUser(String username, String email) {
        usernameInput.fill(username);
        emailInput.fill(email);
        submitBtn.click();
    }
}
```

---

## 3. Key Takeaways

1. **Default Values**: Provide sensible default values inside the `Builder` to minimize setup code in test methods.
2. **Immutable Objects**: Make fields `private final` so test data cannot be altered during execution.

