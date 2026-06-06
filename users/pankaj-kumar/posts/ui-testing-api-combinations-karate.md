---
title: Karate UI Testing & API Combinations
date: 14-Mar-2025
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [karate, ui-testing, chrome, hybrid, automation]
category: API Testing
categories: [API Testing, BDD, Java, Automation, UI Testing]
excerpt: >-
  Ditch Selenium! Learn how to combine lightning-fast API calls for test setup with native Chrome UI automation in a single Karate script.
readTime: 4 min read
---

# Blog #8: Karate UI Testing & API Combinations

Welcome to Part 8 of the **Karate API Testing Series**!

Have you ever written a Selenium UI test where the setup phase (creating a test user) takes 45 seconds of clicking through registration forms? That is extremely inefficient. 

What if you could instantly create the test data via backend APIs in 100 milliseconds, and *then* launch the browser to test the UI? Karate natively supports both API and UI testing in the same script without needing Selenium WebDriver! It uses the Chrome DevTools Protocol under the hood.

## 1. The Hybrid API & UI Script
Let's create `ui.feature` to demonstrate this incredible capability:

```gherkin
# src/test/java/com/mycodeyatra/karate/ui/ui.feature
Feature: Karate UI and API Combination
  Background:
    # Configure Karate to natively use Headless Chrome
    * configure driver = { type: 'chrome', headless: true }
    * url baseUrl
  Scenario: Create a user via API and verify via UI
    # ---------------------------------------------------------
    # 1. API Step: Instantly Setup Data via Backend
    # ---------------------------------------------------------
    Given path '/users'
    And request { name: 'UI Ninja', email: 'ui@karate.com', role: 'admin' }
    When method post
    Then status 201
    * def createdId = response.id
    # ---------------------------------------------------------
    # 2. UI Step: Launch Browser & Verify Frontend
    # ---------------------------------------------------------
    # The 'driver' keyword instantly launches Chrome!
    Given driver 'https://example.com'
    And delay(1000)
    Then match driver.title == 'Example Domain'
    # In a real frontend, you would do something like this:
    # And input('#username', 'ui@karate.com')
    # And click('#login-button')
    # And match text('#welcome-message') contains 'UI Ninja'
```

## 2. The Execution Flow
When you run this hybrid script via Maven, Karate effortlessly bridges the gap between backend HTTP clients and frontend browser automation. 

```bash
mvn clean test -Dtest=UiRunner
[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 19.28 s - in com.mycodeyatra.karate.ui.UiRunner
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
```

By combining APIs for test setup/teardown and UI for critical user journeys, your end-to-end testing time drops from minutes to mere seconds. Karate makes hybrid testing a joy!

Next up, we will tackle **Parallel Execution & Generating Cucumber Reports**!
