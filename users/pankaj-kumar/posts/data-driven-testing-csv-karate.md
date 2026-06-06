---
title: Data Driven Testing with CSVs
date: 11-Mar-2025
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [karate, bdd, data-driven, csv, testing]
category: API Testing
categories: [API Testing, BDD, Java, Automation]
excerpt: >-
  Master Data-Driven Testing in Karate! Learn how to dynamically execute Scenarios using external CSV datasets without writing a single line of parsing code.
readTime: 4 min read
---

# Blog #5: Data Driven Testing with CSVs in Karate

Welcome to Part 5 of the **Karate API Testing Series**!

A robust test framework doesn't hardcode variables; it reads them from external data sources. When testing an API, you often need to run the exact same `Scenario` multiple times with different payloads (e.g., creating a regular user, an admin user, and a user with missing fields).

Karate makes Data-Driven Testing incredibly elegant by natively parsing `.csv` files directly in a `Scenario Outline`.

## 1. Creating the Dataset
First, we create a simple `users.csv` file inside our test folder:

```csv
name,email,role,expectedStatus
Alice,alice@karate.com,admin,201
Bob,bob@karate.com,user,201
Charlie,,user,400
```
*(Notice that Charlie is missing an email, so we expect the API to return a `400 Bad Request`.)*

## 2. Writing the Scenario Outline
In our feature file, we use the `Scenario Outline` syntax. Instead of manually writing out an Examples table, we simply tell Karate to read the CSV file!

```gherkin
# src/test/java/com/mycodeyatra/karate/datadriven/datadriven.feature
Feature: Data Driven Testing with CSV
  Background:
    * url baseUrl
  Scenario Outline: Create users dynamically from CSV
    Given path '/users'
    # Karate automatically replaces <name>, <email>, and <role> with the CSV values!
    And request { name: '<name>', email: '<email>', role: '<role>' }
    When method post
    # We dynamically assert the status code based on the CSV row
    Then status <expectedStatus>
    Examples:
      | read('users.csv') |
```

## 3. The Test Execution
When we run our `DataDrivenRunner` through Maven, Karate reads the CSV and automatically generates and executes **three independent tests**—one for each row in the CSV file!

Here is the execution log proving all 3 scenarios passed dynamically:

```bash
mvn clean test -Dtest=DataDrivenRunner
[INFO] Tests run: 3, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 6.91 s - in com.mycodeyatra.karate.datadriven.DataDrivenRunner
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
```

Karate's CSV binding is lightweight, requires zero POJOs, and zero CSV parsing libraries (no OpenCSV or Apache POI required!).

In our next blog, we will tackle **Reading JSON Payloads from Files** to handle massive request bodies!
