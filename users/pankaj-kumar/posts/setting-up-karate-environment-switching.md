---
title: Setting up Karate and Environment Switching
date: 08-Mar-2025
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [karate, bdd, java, testing, config]
category: API Testing
categories: [API Testing, BDD, Java, Automation]
excerpt: >-
  Learn how to structure your Karate repository for the real world! Configure dynamic environments using karate-config.js and execute folder-specific test runners.
readTime: 5 min read
---

# Blog #2: Setting up Karate and Environment Switching

Welcome to Part 2 of the **Karate API Testing Series**! After writing our first "Hello World" feature file, it's time to structure our repository for the real world.

In enterprise projects, you rarely test against just one environment. You have a local mock server, a `QA` server, a `Staging` server, and a `Production` server. Hardcoding URLs in your tests is a recipe for disaster.

## 1. Dynamic Environment Configurations
Karate handles multi-environment switching natively via `karate-config.js`. This file is executed precisely once before any tests run.

Let's modify our `karate-config.js` to dynamically alter the `baseUrl` depending on the system property passed by Maven:

```javascript
// src/test/java/karate-config.js
function fn() {
  var env = karate.env; // get system property 'karate.env'
  karate.log('karate.env system property was:', env);
  if (!env) {
    env = 'dev'; // default to dev if nothing is passed
  }
  var config = {
    env: env,
    baseUrl: 'http://localhost:8080/api'
  };
  if (env == 'qa') {
    config.baseUrl = 'https://qa.myapi.com/api';
  } else if (env == 'e2e') {
    config.baseUrl = 'https://e2e.myapi.com/api';
  }
  return config;
}
```

## 2. Using the Configuration
Now, inside our Karate feature files, we no longer need to type out `http://localhost:8080`. We simply inject the `baseUrl` variable!

```gherkin
# src/test/java/com/mycodeyatra/karate/setup/setup.feature
Feature: Exploring Advanced Karate Setup
  Scenario: Testing environment variables
    # The 'env' variable is automatically populated by karate-config.js
    * print 'Current environment is:', karate.env
    * print 'Base URL is set to:', baseUrl
    Given url baseUrl + '/health'
    When method get
    Then status 200
    And match response.status == 'OK'
```

## 3. Creating a Custom Runner
To execute tests within specific folders, we create a JUnit 5 runner using `@Karate.Test`:

```java
// src/test/java/com/mycodeyatra/karate/setup/SetupRunner.java
package com.mycodeyatra.karate.setup;
import com.intuit.karate.junit5.Karate;
class SetupRunner {
    @Karate.Test
    Karate testSetup() {
        return Karate.run("setup").relativeTo(getClass());
    }
}
```

## 4. Maven Execution
When executing the tests via Maven, you can pass the environment flag using `-Dkarate.env=qa` to switch your base URLs on the fly!

```bash
mvn clean test -Dtest=SetupRunner -Dkarate.env=dev
[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 7.719 s - in com.mycodeyatra.karate.setup.SetupRunner
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
```

In the next blog, we will master **JSON and XML Expressions** natively within Karate!
