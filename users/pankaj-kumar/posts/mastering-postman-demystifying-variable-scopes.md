---
title: Mastering Postman: Demystifying Variable Scopes
date: 06-Jan-2025
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [postman, api-testing, automation, variables]
category: API Testing
categories: [Api Testing, Postman, Automation]
excerpt: >-
  Mastering Postman: Demystifying Variable Scopes  > Key insight: Using the wrong variable scope can lead to leaked tokens, cross-environment pollution, and incredibly hard-to-debug failures. Knowing *w
readTime: 2 min read
---
> **Important Update:** To get the most out of this tutorial, we highly recommend running the official [MyCodeYatra Mock API Server](https://github.com/MYCodeYatra/myct-api-test-server) locally on `http://localhost:8080`. Replace any generic public API URLs in these examples with your local Mock Server endpoints!



# Mastering Postman: Demystifying Variable Scopes

> **Key insight:** Using the wrong variable scope can lead to leaked tokens, cross-environment pollution, and incredibly hard-to-debug failures. Knowing *where* to store your data is just as important as knowing *how* to store it.

In our previous discussions, we extracted tokens and saved them as Environment Variables. But Postman actually offers five different levels of variable scope. 

If you've ever spent an hour wondering why your request is using an old token from yesterday, it is almost certainly a scoping issue.

## 1. The Scope Hierarchy

Postman resolves variables from the narrowest scope to the broadest scope. If two variables share the same name, the narrower scope wins.


![diagram_1](https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/mastering-postman-demystifying-variable-scopes/images/diagram_1.png)


### 1. Global Variables
*   **What they are:** Variables accessible across *all* collections and workspaces.
*   **When to use them:** Rarely. Only use them for universally constant data (e.g., a specific test user's email that never changes regardless of environment).
*   **Syntax:** `pm.globals.set("name", "value")`

### 2. Collection Variables
*   **What they are:** Variables specific to a single Collection.
*   **When to use them:** For static configuration data that applies to the whole collection, regardless of environment (e.g., `api_version: v2` or endpoint paths).
*   **Syntax:** `pm.collectionVariables.set("name", "value")`

### 3. Environment Variables
*   **What they are:** Sets of variables bound to a specific context (Dev, QA, Prod).
*   **When to use them:** For anything that changes based on where you are pointing your requests (e.g., `base_url`, `db_password`, `auth_token`).
*   **Syntax:** `pm.environment.set("name", "value")`

### 4. Data Variables
*   **What they are:** Variables pulled from external CSV or JSON files when running the Collection Runner.
*   **When to use them:** For data-driven testing (testing the same endpoint with 100 different rows of data).

### 5. Local Variables
*   **What they are:** Temporary variables that only exist for the duration of a single script execution.
*   **When to use them:** To store intermediate calculations that you don't want permanently saved to your environment.
*   **Syntax:** `pm.variables.set("name", "value")`

## 2. A Real-World Scoping Conflict

Imagine you have an Environment variable named `token` = `abc`.
You then write a Pre-request script:
```javascript
pm.variables.set("token", "xyz");
```

When you send a request with `Authorization: Bearer {{token}}`, which token is sent?

Because `pm.variables.set()` creates a **Local Variable**, it overrides the Environment variable. The request will send `xyz`. However, once the request finishes, the Local variable disappears, and your Environment variable remains `abc` untouched!

## Final Takeaways

Always use the narrowest scope possible to prevent data leakage. Default to **Local** variables for temporary script calculations, **Environment** variables for auth tokens and URLs, and **Collection** variables for static paths. In our next phase, we will step up our game and look at OAuth 2.0 token automation!
