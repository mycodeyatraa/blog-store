---
title: Scenario Context, State Sharing & Tags - Playwright Java Design Patterns
date: 26-Feb-2026
lastUpdated: 26-Feb-2026
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
  Share test context between steps using PicoContainer DI and filter test runs with Cucumber tags in Playwright Java.
readTime: 9 min read
---

# Scenario Context, State Sharing & Tags - Playwright Java Design Patterns

Sharing variables (such as created order IDs, authentication tokens, or page instances) across step definition classes requires Dependency Injection.

PicoContainer is the standard lightweight DI framework for Cucumber-JVM.

---

## 1. Scenario Context Sharing via PicoContainer

```java
public class ScenarioContext {
    private String createdUserId;
 
    public String getCreatedUserId() { return createdUserId; }
    public void setCreatedUserId(String id) { this.createdUserId = id; }
}
```

---

## 2. Tagged Test Execution

```gherkin
@smoke @regression
Scenario: Verify Quick Add to Cart
  Given the user is logged in
  When user adds product to cart
  Then product count should update
```

```bash
# Execute only @smoke tagged scenarios
mvn test -Dcucumber.filter.tags="@smoke"
```

---

## 3. Key Takeaways

1. **Zero Global Static Variables**: Share state cleanly via PicoContainer dependency injection.
2. **Flexible Tagging**: Filter test execution in CI/CD pipelines using logical tag expressions (`@smoke and not @flaky`).

