---
title: Reading JSON Payloads from Files
date: 12-Mar-2025
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [karate, bdd, json, payload, automation]
category: API Testing
categories: [API Testing, BDD, Java, Automation]
excerpt: >-
  Keep your Karate feature files clean! Learn how to read external JSON files natively, dynamically mutate their contents, and use them as API request payloads.
readTime: 4 min read
---

# Blog #6: Reading JSON Payloads from Files in Karate

Welcome to Part 6 of the **Karate API Testing Series**!

In real-world enterprise applications, API request payloads can be massive. You do not want to clutter your clean `.feature` files with 500 lines of raw JSON.

Karate provides a built-in `read()` function that allows you to seamlessly load external JSON files into your test context and instantly use them as request payloads.

## 1. Creating the JSON Payload
First, we create a JSON file named `user_payload.json` inside our test directory:

```json
{
  "name": "Payload Ninja",
  "email": "payload@karate.com",
  "role": "admin"
}
```

## 2. Reading and Mutating the Payload
Now, in our feature file, we can read this JSON file into a variable. 

But wait, what if we need to dynamically change the `name` before sending the request? Karate natively allows you to mutate JSON variables on the fly using the `set` keyword!

```gherkin
# src/test/java/com/mycodeyatra/karate/payloads/payloads.feature
Feature: Reading JSON Payloads from Files
  Background:
    * url baseUrl
  Scenario: Create user using an external JSON file
    # 1. Read the JSON file into a native JSON object
    * def requestBody = read('user_payload.json')
    # 2. Dynamically modify the payload!
    * set requestBody.name = 'Modified Payload Ninja'
    # 3. Send the modified payload
    Given path '/users'
    And request requestBody
    When method post
    Then status 201
    # 4. Assert the dynamically modified value was successfully saved
    And match response.name == 'Modified Payload Ninja'
    And match response.email == 'payload@karate.com'
    And match response.id == '#uuid'
```

## 3. The Test Execution
When we run our `PayloadRunner` through Maven, Karate parses the JSON, dynamically modifies the requested fields in memory, and asserts the successful response.

```bash
mvn clean test -Dtest=PayloadRunner
[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 10.218 s - in com.mycodeyatra.karate.payloads.PayloadRunner
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
```

By externalizing your payloads into `.json` files, your tests remain readable, and you can reuse payloads across multiple scenarios by simply mutating them before the request!

Stay tuned for our next blog: **Type Conversion & Data Generators in Karate!**
