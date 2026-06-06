---
title: Introduction to Karate DSL
date: 07-Mar-2025
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [karate, bdd, java, api-testing]
category: API Testing
categories: [API Testing, BDD, Automation, Java]
excerpt: >-
  Welcome to the Karate API Testing Series! Enter the realm of Behavior Driven Development (BDD) with Karate DSL and write your very first feature file.
readTime: 4 min read
---

# Blog #1: Introduction to Karate DSL

Welcome to the **Karate API Testing Series**! After conquering Postman, RestAssured, and SuperTest, we are now entering the realm of **Behavior Driven Development (BDD)** with Karate DSL.

Karate is unique because it doesn't require you to write Java code to test your APIs. Instead, you write tests in a domain-specific language (DSL) based on Gherkin (Given/When/Then), but supercharged with native JSON and XML support.

## 1. Setting up the Project
First, we create a basic Maven project and include the `karate-junit5` dependency in our `pom.xml`:

```xml
<dependency>
    <groupId>com.intuit.karate</groupId>
    <artifactId>karate-junit5</artifactId>
    <version>1.4.1</version>
    <scope>test</scope>
</dependency>
```

## 2. Global Configuration
Karate uses a JavaScript file named `karate-config.js` to set up environment variables globally across all your feature files. Let's point it to the ReqRes mock API:

```javascript
// src/test/java/karate-config.js
function fn() {
  return {
    baseUrl: 'https://reqres.in/api'
  };
}
```

## 3. Our First Feature File
Instead of creating a Java class, we create a `.feature` file. Notice how clean and readable the syntax is. We don't need to parse JSON objects or write complex assertions—Karate understands JSON natively!

```gherkin
# src/test/java/com/mycodeyatra/karate/intro/hello.feature
Feature: Introduction to Karate DSL
  Background:
    * url baseUrl
  Scenario: Verify basic API GET request
    Given path '/users/2'
    When method get
    Then status 200
    And match response.data.id == 2
    And match response.data.first_name == 'Janet'
```

## 4. The JUnit 5 Runner
To execute this feature file, we only need a tiny 5-line Java class:

```java
package com.mycodeyatra.karate.intro;
import com.intuit.karate.junit5.Karate;
class IntroRunner {
    @Karate.Test
    Karate testIntro() {
        return Karate.run("hello").relativeTo(getClass());
    }
}
```

## 5. Test Execution
When we run `mvn clean test`, Karate automatically parses the `.feature` file, executes the HTTP request, and strictly asserts the JSON response.

```bash
[INFO] --- maven-surefire-plugin:3.0.0-M7:test (default-test) @ mcyt-api-karate ---
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  40.079 s
```

In the next blog, we will dive deeper into **Setting up Karate** for enterprise-scale frameworks, handling environment switching, and organizing folder structures!
