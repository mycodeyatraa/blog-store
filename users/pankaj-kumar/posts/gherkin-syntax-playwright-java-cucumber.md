---
title: "Mastering Gherkin Syntax in Playwright Java Frameworks"
date: "11-Aug-2026"
lastUpdated: "11-Aug-2026"
author: "pankaj-kumar"
authorName: "Pankaj Kumar"
authorRole: "Automation Architect"
authorAvatar: "https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG"
authorBio: "Automation Architect"
authorGithub: "https://github.com/pankajhyd"
authorLinkedin: "https://www.linkedin.com/in/pankaj-kumar-94a2b227/"
tags: ["Playwright", "Java", "Gherkin", "Cucumber", "Syntax"]
category: "Automation Testing"
categories: ["Automation Testing", "Playwright Java", "Gherkin"]
excerpt: "Learn how to write unambiguous Gherkin feature specifications using Given, When, Then, And, and But keywords for Playwright Java automation."
readTime: "7 min read"
---

# Mastering Gherkin Syntax in Playwright Java Frameworks

Gherkin is the Business Readable, Domain Specific Language (DSL) that allows software behavior to be described without tying it to implementation details.

---

## 1. Essential Gherkin Keywords

- **Feature**: High-level title and description of the application functionality.
- **Background**: Precondition steps executed before *every* scenario in a feature file.
- **Scenario**: A concrete user flow or business rule validation.
- **Given**: Initial context or system state (preconditions).
- **When**: Action taken by the user or system event.
- **Then**: Expected outcome or assertion verification.
- **And / But**: Conjunctions for combining multiple preconditions, actions, or outcomes.

---

## 2. Advanced Feature Specification

```gherkin
# src/test/resources/features/shopping_cart.feature
Feature: E-Commerce Shopping Cart Management
 
  Background:
    Given the user is logged into the application with valid account
    And the shopping cart is empty
 
  Scenario: Add single product to shopping cart
    When the user searches for product "Wireless Mouse"
    And clicks on "Add to Cart" button for item "WM-100"
    Then the shopping cart count should display "1"
    And the item "Wireless Mouse" should be present in cart list
    But no discount code should be applied automatically
```

---

## 3. Playwright Java Glue Code Implementation

```java
// src/test/java/com/mycodeyatra/steps/ShoppingCartSteps.java
package com.mycodeyatra.steps;
 
import com.microsoft.playwright.*;
import io.cucumber.java.en.*;
import static org.junit.jupiter.api.Assertions.*;
 
public class ShoppingCartSteps {
    private Page page;
 
    public ShoppingCartSteps(Hooks hooks) {
        this.page = hooks.getPage();
    }
 
    @Given("the user is logged into the application with valid account")
    public void loginUser() {
        page.navigate("https://mycodeyatra.com/login");
        page.fill("#username", "qa_user");
        page.fill("#password", "password123");
        page.click("#login-btn");
    }
 
    @Given("the shopping cart is empty")
    public void verifyEmptyCart() {
        page.click("#cart-icon");
        assertTrue(page.isVisible(".empty-cart-message"));
    }
 
    @When("the user searches for product {string}")
    public void searchProduct(String productName) {
        page.fill("#search-input", productName);
        page.keyboard().press("Enter");
    }
 
    @When("clicks on {string} button for item {string}")
    public void clickAddToCart(String buttonText, String sku) {
        page.click(String.format("button[data-sku='%s']", sku));
    }
 
    @Then("the shopping cart count should display {string}")
    public void verifyCartCount(String expectedCount) {
        assertEquals(expectedCount, page.textContent("#cart-badge"));
    }
 
    @Then("the item {string} should be present in cart list")
    public void verifyItemInCart(String itemName) {
        page.click("#cart-icon");
        assertTrue(page.isVisible(String.format("text=%s", itemName)));
    }
 
    @Then("no discount code should be applied automatically")
    public void verifyNoDiscount() {
        assertFalse(page.isVisible(".discount-applied-badge"));
    }
}
```

---

## 4. Best Practice Comparison

| Practice | Bad Gherkin (Implementation Coupled) | Good Gherkin (Declarative Behavior) |
| :--- | :--- | :--- |
| **Login** | When I type "user" in input `#usr` and click button `#btn` | When the user submits valid login credentials |
| **Navigation** | Given I open page `http://localhost/app/home.html` | Given the user is on the home dashboard page |
| **Assertions** | Then div `.alert-success` should have css color `#00ff00` | Then a success confirmation toast should be visible |
