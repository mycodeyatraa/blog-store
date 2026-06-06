---
title: Calling other Feature Files
date: 10-Mar-2025
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [karate, bdd, dry, authentication, framework]
category: API Testing
categories: [API Testing, BDD, Java, Automation]
excerpt: >-
  Keep your testing codebase DRY! Learn how to create reusable authentication scripts in Karate and dynamically pass variables between multiple feature files.
readTime: 4 min read
---

# Blog #4: Calling other Feature Files in Karate

Welcome to Part 4 of the **Karate API Testing Series**! As your test automation framework grows, you will inevitably encounter operations that must be executed across dozens of test scenarios. 

The most common example? **Authentication.**

If every test script duplicates the login logic, a single change to the auth payload will break your entire framework. Fortunately, Karate provides an incredibly elegant solution: **Feature Files calling other Feature Files.**

## 1. Creating a Reusable Feature File
Let's create a reusable script called `login.feature`. 

Notice the `@ignore` tag at the top. This tells Karate's JUnit runner *not* to execute this file independently—it should only be invoked by another feature file.

```gherkin
# src/test/java/com/mycodeyatra/karate/reusable/login.feature
@ignore
Feature: Reusable Login Feature
  Scenario: Perform login and return token
    # We expect 'userEmail' and 'userPassword' to be injected by the caller
    Given url baseUrl + '/auth/login'
    And request { email: '#(userEmail)', password: '#(userPassword)' }
    When method post
    Then status 200
    # Store the returned JWT token as a variable so the caller can read it!
    * def authToken = response.token
```

## 2. Calling the Reusable Script dynamically
Now, let's create our main test: `caller.feature`. 

We will use the `call read('...')` syntax to invoke `login.feature`, passing in our dynamic variables as a JSON payload.

```gherkin
# src/test/java/com/mycodeyatra/karate/reusable/caller.feature
Feature: Calling Other Feature Files
  Background:
    * url baseUrl
  Scenario: Fetch protected profile using a token from another feature
    # 1. Call the login script and pass the required credentials
    * def loginResult = call read('login.feature') { userEmail: 'admin@mycodeyatra.com', userPassword: 'password123' }
    # 2. Extract the generated token from the result
    * def myToken = loginResult.authToken
    * print 'Successfully retrieved token:', myToken
    # 3. Inject the token into the Authorization header of the next request!
    Given path '/auth/profile'
    And header Authorization = 'Bearer ' + myToken
    When method get
    Then status 200
    And match response.message == 'Welcome to your protected profile!'
```

## 3. The Execution Flow
When you run `CallerRunner` using Maven, Karate seamlessly branches into `login.feature`, injects the `userEmail` and `userPassword`, parses the HTTP 200 response, saves the token, returns control back to `caller.feature`, and injects the token into the `/auth/profile` request. 

This creates extremely modular, maintainable, and DRY (Don't Repeat Yourself) codebases.

Stay tuned for our next post where we will tackle **Karate UI Testing and API combinations!**
