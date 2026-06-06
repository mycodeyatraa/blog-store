---
title: Type Conversion & Data Generators
date: 13-Mar-2025
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [karate, java, bdd, interoperability, generators]
category: API Testing
categories: [API Testing, BDD, Java, Automation]
excerpt: >-
  Bypass API uniqueness constraints! Learn how to seamlessly invoke Java code directly from your Karate scripts to generate random data, and master native type conversions.
readTime: 4 min read
---

# Blog #7: Type Conversion & Data Generators in Karate

Welcome to Part 7 of the **Karate API Testing Series**!

One of the hardest parts of API testing is dealing with **Uniqueness Constraints**. If your API requires a unique email address to register a user, hardcoding `test@email.com` will work exactly *once*. On the second test run, it will fail with a `409 Conflict`. 

To solve this, Karate natively supports Java interoperability, allowing you to invoke custom Java code directly from your test scripts! We'll also cover Type Conversions—a common stumbling block when reading numerical data from flat text files.

## 1. Creating a Java Data Generator
First, we'll create a simple Java class with a static method that spits out unique email addresses using `UUID.randomUUID()`.

```java
// src/test/java/com/mycodeyatra/karate/generators/DataGenerator.java
package com.mycodeyatra.karate.generators;
import java.util.UUID;
public class DataGenerator {
    public static String getRandomEmail() {
        return "user_" + UUID.randomUUID().toString().substring(0, 8) + "@mycodeyatra.com";
    }
}
```

## 2. Using Java Interop and Type Conversion in Karate
Now, in our feature file, we use the `Java.type(...)` function to import our class. We can immediately invoke the `getRandomEmail()` method and assign it to a variable!

We will also demonstrate how to cast strings into integers using the built-in `parseInt` function (or Karate's shortcut `~~`).

```gherkin
# src/test/java/com/mycodeyatra/karate/generators/generators.feature
Feature: Type Conversion & Data Generators
  Background:
    * url baseUrl
  Scenario: Generate random email and perform type conversion
    # 1. Calling Java from Karate to generate a random string!
    * def DataGen = Java.type('com.mycodeyatra.karate.generators.DataGenerator')
    * def randomEmail = DataGen.getRandomEmail()
    # 2. Type Conversion: String to Number
    * def stringAge = '25'
    * def intAge = parseInt(stringAge)
    * match intAge == 25
    # 3. Using the generated data in an API call
    Given path '/users'
    And request { name: 'Dynamic User', email: '#(randomEmail)', age: '#(intAge)', role: 'user' }
    When method post
    Then status 201
    # Asserting the dynamically generated data was accepted
    And match response.email == randomEmail
```

## 3. The Test Execution
When we run our `GeneratorRunner` using Maven, Karate seamlessly executes the Java bytecode, injects the dynamic string into our JSON request payload, parses the String age into an Integer, and validates the response natively.

```bash
mvn clean test -Dtest=GeneratorRunner
[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 9.606 s - in com.mycodeyatra.karate.generators.GeneratorRunner
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
```

By leveraging Java interop, your Karate scripts possess the full power and ecosystem of Java while retaining the clean readability of BDD Gherkin!

In our next tutorial, we will finally transition to **Karate UI Testing and API combinations**!
