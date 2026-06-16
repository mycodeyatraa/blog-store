---
title: The Backend Pivot: API Testing with Axios
date: 01-Dec-2024
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [typescript, selenium, api-testing, axios, supertest]
category: Selenium TypeScript
categories: [Selenium TypeScript, API Testing Integration]
excerpt: >-
  UI tests are slow. Shift-left your testing pyramid by integrating Axios to perform lightning-fast API automation in TypeScript.
readTime: 4 min read
---

# The Backend Pivot: API Testing with Axios

Welcome to **Phase 5: API Integration & Backend Testing**!

Until now, we have exclusively focused on the User Interface (UI). While UI automation is vital, it is notoriously slow and flaky. Modern Test Automation Pyramids suggest that 70% of your testing should occur at the API layer.

Furthermore, we often need to "setup" data before a UI test runs. Instead of slowly clicking through a UI to create a user, what if we just sent an instant API `POST` request to generate the user, and *then* opened the browser to verify them?

To do this in TypeScript, we need an HTTP Client. The two most popular choices are **Axios** and **SuperTest**. 

In this framework, we will use **Axios**.

---

## 1. Installing Axios

Axios is a promise-based HTTP client that natively supports TypeScript. 

Install it in your framework:

```bash
npm install axios
```

---

## 2. Your First API Test

Let's write a simple script that hits a public REST API (`jsonplaceholder.typicode.com`), fetches a user, and asserts the HTTP Status Code and payload.

Create `tests/api_axios_basics.test.ts`:

```typescript
import axios from "axios";
describe("Phase 5 - API Testing: Axios Basics", () => {
  it("Should perform a basic GET request using Axios", async () => {
    // 1. Send the GET request to a public mock API
    const response = await axios.get("https://jsonplaceholder.typicode.com/users/1");
    // 2. Validate the Response Status Code (200 OK)
    expect(response.status).toBe(200);
    // 3. Validate the Response Payload (Body)
    const userData = response.data;
    console.log(`Successfully fetched user: ${userData.name}`);
    expect(userData.id).toBe(1);
    expect(userData.email).toBeDefined();
  });
  it("Should handle API errors gracefully", async () => {
    try {
      // Sending a request to an endpoint that does not exist (404)
      await axios.get("https://jsonplaceholder.typicode.com/users/9999");
    } catch (error: any) {
      // Axios inherently throws a JavaScript error on 4xx and 5xx responses!
      expect(error.response.status).toBe(404);
      console.log(`Caught expected error: ${error.message}`);
    }
  });
});
```

---

## 3. Test Execution Output

When we run this test:

```bash
> jest tests/api_axios_basics.test.ts
```

We see how incredibly fast API tests are compared to spinning up a ChromeDriver!

```text
 PASS  tests/api_axios_basics.test.ts
  Phase 5 - API Testing: Axios Basics
    √ Should perform a basic GET request using Axios (152 ms)
    √ Should handle API errors gracefully (65 ms)
  console.log
    Successfully fetched user: Leanne Graham
  console.log
    Caught expected error: Request failed with status code 404
```

Notice the execution time: **152 ms**. That is the true power of API testing!

## Conclusion

Axios is incredibly powerful, allowing us to send network requests directly from our Jest test scripts.

However, in this first test, we relied on hardcoded URLs. In our next tutorial, we will formalize our API structure and dive deep into **GET APIs**, learning how to extract dynamic Base URLs from our `.env` files to build a scalable API framework.
