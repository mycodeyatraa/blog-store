---
title: Organizing Feature Files and Cucumber Configurations
date: 18-Apr-2025
lastUpdated: 18-Apr-2025
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: ["playwright", "typescript", "bdd", "cucumber", "architecture"]
category: UI Automation
categories: ["UI Automation", "Playwright", "TypeScript", "BDD", "Cucumber"]
excerpt: >-
  Learn enterprise best practices for structuring your BDD Feature Files and mapping them dynamically using cucumber.js configurations.
readTime: 4 min read
---

When building an enterprise BDD framework, how you organize your `.feature` files is just as important as the code inside them. A disorganized features directory will quickly become a nightmare for Product Managers and QA Engineers to navigate.

In this tutorial, we will explore best practices for structuring your Feature Files and configuring Cucumber to execute them effectively.

### Structuring Your Features Directory

Feature files should map directly to the business domains or epics of your application. Avoid dumping all `.feature` files into a single flat directory. Instead, use nested folders to represent specific modules.

```text
📦 mcyt-plw-typescript
 ┣ 📂 features
 ┃ ┣ 📂 authentication
 ┃ ┃ ┣ 📜 login.feature
 ┃ ┃ ┣ 📜 password_reset.feature
 ┃ ┃ ┗ 📜 registration.feature
 ┃ ┣ 📂 shopping_cart
 ┃ ┃ ┣ 📜 add_to_cart.feature
 ┃ ┃ ┗ 📜 checkout.feature
 ┃ ┗ 📂 user_profile
 ┃   ┗ 📜 update_settings.feature
```

This structure makes it incredibly easy for a stakeholder to drill down into the exact functionality they want to review.

### Configuring Cucumber.js

To tell the test runner where these meticulously organized files live, we must create a configuration file at the root of our project. 

**File:** `cucumber.js`

```javascript
module.exports = {
  default: {
    // Specify the path to where your feature files live using glob patterns
    paths: ['features/**/*.feature'],
 
    // Specify where your step definitions are located
    require: ['tests/steps/**/*.ts'],
 
    // Load TypeScript compiler for Cucumber natively
    requireModule: ['ts-node/register'],
 
    // Output formats: output to console and generate a JSON report
    format: [
      'progress-bar',
      'json:reports/cucumber-report.json'
    ],
 
    // Default parallel execution for faster test runs
    parallel: 2
  }
};
```

### The Power of Glob Patterns

In the `paths` array, we utilize the glob pattern `'features/**/*.feature'`. This tells Cucumber to recursively scan the `features/` directory and every single sub-directory within it, automatically picking up and queuing any file ending in `.feature`.

If you ever want to run a specific module exclusively—for example, only the shopping cart tests—you can easily override this via the CLI:

```bash
npx cucumber-js features/shopping_cart/**/*.feature
```

### Summary

Treat your `.feature` files as living documentation. By organizing them logically into business domains and pointing your `cucumber.js` configuration to the root folder, your repository transforms into a searchable, structured database of application requirements that execute natively against your Playwright code!
