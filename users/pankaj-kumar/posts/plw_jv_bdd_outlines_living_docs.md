---
title: Scenario Outlines, Data Tables & Living Docs - Playwright Java Design Patterns
date: 27-Feb-2026
lastUpdated: 27-Feb-2026
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
  Run data-driven BDD tests with Scenario Outlines and generate rich HTML living documentation in Playwright Java.
readTime: 9 min read
---

# Scenario Outlines, Data Tables & Living Docs - Playwright Java Design Patterns

Scenario Outlines allow running the same BDD scenario repeatedly across multiple data sets defined in an `Examples:` table.

---

## 1. Scenario Outline Data-Driven Test

```gherkin
Scenario Outline: User Login Validation
  Given user opens login page
  When user enters "<username>" and "<password>"
  Then status message should be "<expected_status>"
 
  Examples:
    | username   | password   | expected_status |
    | admin      | secret123  | Success         |
    | invalid    | wrongpass  | Error           |
    | locked     | pass123    | Locked Out      |
```

---

## 2. Generating HTML Living Documentation

```xml
<!-- pom.xml Cucumber HTML Plugin Config -->
<plugin>
    <groupId>net.masterthought</groupId>
    <artifactId>maven-cucumber-reporting</artifactId>
    <version>5.7.0</version>
</plugin>
```

---

## 3. Key Takeaways

1. **Data-Driven BDD**: Eliminate code duplication by parameterizing scenario steps.
2. **Rich HTML Reports**: Publish interactive living documentation for stakeholders after every CI run.

