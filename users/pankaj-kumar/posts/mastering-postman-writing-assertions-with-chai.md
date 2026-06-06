---
title: Mastering Postman: Writing Assertions with Chai
date: 04-Jan-2025
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [postman, api-testing, automation, assertions]
category: API Testing
categories: [Api Testing, Postman, Automation]
excerpt: >-
  Mastering Postman: Writing Assertions with Chai  > Key insight: Hitting an endpoint and getting a `200 OK` means the server didn't crash, but it doesn't mean your API is actually working. True automat
readTime: 2 min read
---
> **Important Update:** To get the most out of this tutorial, we highly recommend running the official [MyCodeYatra Mock API Server](https://github.com/MYCodeYatra/myct-api-test-server) locally on `http://localhost:8080`. Replace any generic public API URLs in these examples with your local Mock Server endpoints!



# Mastering Postman: Writing Assertions with Chai

> **Key insight:** Hitting an endpoint and getting a `200 OK` means the server didn't crash, but it doesn't mean your API is actually working. True automation requires strict programmatic assertions.

In the previous post, we used Pre-request scripts to dynamically generate data. Now it's time to validate what the server sends back. 

If you are manually looking at the JSON response body to verify that `"status": "success"`, you are wasting valuable time. Postman includes the **Chai Assertion Library** built-in, allowing you to write powerful, readable tests in JavaScript.

## 1. The Anatomy of a Postman Test

All assertions in Postman are written in the **Tests** tab. These scripts execute *after* the network request completes and the response is received.


![diagram_1](https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/mastering-postman-writing-assertions-with-chai/images/diagram_1.png)


A standard Postman test block uses the `pm.test` function, which takes two arguments:
1. A human-readable name for the test.
2. A callback function containing the actual assertions.

```javascript
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});
```

## 2. Testing the JSON Body

Checking the status code is the bare minimum. A real test digs into the JSON payload. Postman makes this easy by automatically parsing the response body into a JavaScript object via `pm.response.json()`.

Let's assume our API returns this payload:
```json
{
    "user_id": 1045,
    "email": "testuser@mycodeyatra.com",
    "roles": ["admin", "editor"],
    "is_active": true
}
```

Here is how you would write assertions to thoroughly validate that response:

```javascript
const jsonData = pm.response.json();

pm.test("User is active", function () {
    // Assert boolean value
    pm.expect(jsonData.is_active).to.be.true;
});

pm.test("Email matches domain", function () {
    // Assert string contains
    pm.expect(jsonData.email).to.include("@mycodeyatra.com");
});

pm.test("User has admin role", function () {
    // Assert array inclusion
    pm.expect(jsonData.roles).to.be.an("array").that.includes("admin");
});
```

### Pro-Tip: The "expect" syntax
Chai uses a BDD (Behavior-Driven Development) style syntax via `pm.expect()`. You can chain words together so the test reads almost exactly like English: `.to.be.a("string").that.is.not.empty`.

## 3. Performance Assertions

Functional correctness isn't the only thing that matters. If an endpoint takes 4 seconds to return data, it might as well be broken. 

You should always enforce an SLA (Service Level Agreement) directly in your tests:

```javascript
pm.test("Response time is less than 300ms", function () {
    pm.expect(pm.response.responseTime).to.be.below(300);
});
```

## Final Takeaways

Writing tests inside Postman transforms your collections from simple "API Pingers" into a robust test suite. By asserting status codes, payload structures, and performance metrics, you create a safety net that catches regressions instantly. In the next post, we will look at how to extract data from these responses to chain multiple requests together!
