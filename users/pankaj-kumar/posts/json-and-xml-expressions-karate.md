---
title: JSON and XML Expressions
date: 09-Mar-2025
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [karate, bdd, json, xml, assertions]
category: API Testing
categories: [API Testing, BDD, Java, Automation]
excerpt: >-
  Discover the true power of Karate DSL! Learn how to assert complex JSON objects and deeply nested XML responses natively without writing a single Java POJO.
readTime: 4 min read
---

# Blog #3: JSON and XML Expressions in Karate

Welcome to Part 3 of the **Karate API Testing Series**! Today, we explore what makes Karate truly magical: its native handling of JSON and XML.

If you've used Java frameworks like RestAssured or JUnit, you know the pain of configuring Jackson or Gson, mapping POJOs, or dealing with tedious XML Document Builders. In Karate, you don't need any of that. JSON and XML are first-class citizens.

## 1. Native JSON Assertions
Let's create a test that POSTs a new user and asserts the response. Notice how we write the payload natively within the `.feature` file!

```gherkin
# src/test/java/com/mycodeyatra/karate/expressions/expressions.feature
Feature: JSON and XML Expressions in Karate
  Background:
    * url baseUrl
  Scenario: Native JSON Parsing and Assertions
    # Send a JSON Payload dynamically!
    Given path '/users'
    And request { name: 'Karate Ninja', email: 'ninja@karate.com', role: 'admin' }
    When method post
    Then status 201
    # Asserting JSON fields naturally
    And match response.name == 'Karate Ninja'
    And match response.email == 'ninja@karate.com'
    # Karate's built-in fuzzy matchers!
    And match response.id == '#uuid'
    And match response.createdAt == '#notnull'
```

### Karate Fuzzy Matchers
Did you notice the `#uuid` and `#notnull`? Karate provides "Fuzzy Matchers" out of the box. Since APIs generate dynamic IDs and timestamps, we can't hardcode them in our assertions. Karate allows us to assert the *schema* type instead!

## 2. Native XML Parsing
Now, let's hit an endpoint that returns XML. Karate automatically parses XML into a searchable, JSON-like tree structure. No XPath hacking required!

```gherkin
  Scenario: Native XML Parsing and Assertions
    Given path '/chaos/xml'
    When method get
    Then status 200
    # Karate seamlessly traverses XML nodes using slash notation
    And match response/users/user/name == 'Alice'
    And match response/users/user/id == '1'
```

## 3. Test Execution Results
Running this `ExpressionsRunner` class through Maven yields perfectly clean execution logs.

```bash
[INFO] Tests run: 2, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 10.822 s - in com.mycodeyatra.karate.expressions.ExpressionsRunner
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
```

In just a few lines of code, we validated complex JSON structures and traversed deeply nested XML responses without writing a single line of Java! 

Stay tuned for our next blog, where we learn how to keep our code DRY by **Calling other Feature Files**!
