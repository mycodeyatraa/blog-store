---
title: API Framework Design: POJOs and the Builder Pattern
date: 13-Dec-2024
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [typescript, selenium, api-testing, design-patterns, builder-pattern, pojo]
category: Selenium TypeScript
categories: [Selenium TypeScript, API Testing Integration]
excerpt: >-
  Design a scalable API framework in TypeScript by implementing the Builder Pattern to dynamically construct resilient test data payloads via POJOs.
readTime: 5 min read
---

# API Framework Design: POJOs and the Builder Pattern

As your API test suite grows from 10 tests to 1000 tests, managing JSON payloads becomes a massive headache. Hardcoding JSON objects directly inside your test files violates the DRY (Don't Repeat Yourself) principle and makes test maintenance nearly impossible.

What if the backend adds a new required field to the `/posts` endpoint? You would have to update 1000 individual JSON strings!

In this tutorial, we will architect a scalable API framework using **POJOs (Plain Old Javascript Objects)** and the **Builder Design Pattern**.

---

## 1. What is a POJO?

A POJO in TypeScript is just an interface or class that strictly defines the structure of your data payload. By defining an interface, we get autocomplete, type safety, and a single source of truth for our API contracts.

## 2. What is the Builder Pattern?

The Builder Pattern allows us to construct complex objects step-by-step. Instead of passing massive configuration objects, we chain methods together. It pairs beautifully with POJOs to generate test data dynamically.

---

## 3. Creating the Payload Builder

Create a new file `tests/data/PostPayloadBuilder.ts`:

```typescript
// 1. POJO (Plain Old Javascript Object) Interface
export interface PostPayload {
  title: string;
  body: string;
  userId: number;
}
export class PostPayloadBuilder {
  private payload: PostPayload;
  constructor() {
    // 2. Define a default, fully valid payload
    this.payload = {
      title: "Default Generated Title",
      body: "Default Generated Body Content",
      userId: 1
    };
  }
  // 3. Chainable methods to override specific fields
  public withTitle(title: string): PostPayloadBuilder {
    this.payload.title = title;
    return this;
  }
  public withBody(body: string): PostPayloadBuilder {
    this.payload.body = body;
    return this;
  }
  public withUserId(userId: number): PostPayloadBuilder {
    this.payload.userId = userId;
    return this;
  }
  // 4. Return the final POJO
  public build(): PostPayload {
    return this.payload;
  }
}
```

---

## 4. Writing the Refactored Test Script

Now let's see the Builder Pattern in action. Instead of hardcoding raw JSON strings, our tests will construct payloads dynamically!

Create `tests/api_framework_design.test.ts`:

```typescript
import { ApiUtils } from "./utils/ApiUtils";
import { PostPayloadBuilder } from "./data/PostPayloadBuilder";
describe("Phase 6 - API Framework Design: POJOs and Builders", () => {
  it("Should create a payload using default Builder values", async () => {
    // 1. Generate a default payload using the builder
    const defaultPayload = new PostPayloadBuilder().build();
    // 2. Pass the POJO to the API Utility
    const response = await ApiUtils.post("/posts", defaultPayload);
    // 3. Validate response
    expect(response.status).toBe(201);
    expect(response.data.title).toBe("Default Generated Title");
    console.log(`Created Default Post. Title: ${response.data.title}`);
  });
  it("Should override specific fields using Builder methods", async () => {
    // 1. Generate a custom payload using chained builder methods
    const customPayload = new PostPayloadBuilder()
      .withTitle("Custom Title via Builder")
      .withUserId(99)
      .build();
    // 2. Pass the custom POJO to the API Utility
    const response = await ApiUtils.post("/posts", customPayload);
    // 3. Validate response matches custom overrides
    expect(response.status).toBe(201);
    expect(response.data.title).toBe("Custom Title via Builder");
    expect(response.data.userId).toBe(99);
    // The body should remain the default because we didn't override it!
    expect(response.data.body).toBe("Default Generated Body Content");
    console.log(`Created Custom Post. Title: ${response.data.title}, UserID: ${response.data.userId}`);
  });
});
```

---

## 5. Test Execution Output

Run the test:

```bash
> ENV_NAME=qa jest tests/api_framework_design.test.ts
```

Output:

```text
 PASS  tests/api_framework_design.test.ts
  Phase 6 - API Framework Design: POJOs and Builders
    √ Should create a payload using default Builder values (207 ms)
    √ Should override specific fields using Builder methods (197 ms)
  console.log
    [API] Sending POST request to: https://jsonplaceholder.typicode.com/posts
    [API] Received Status: 201
    Created Default Post. Title: Default Generated Title
  console.log
    [API] Sending POST request to: https://jsonplaceholder.typicode.com/posts
    [API] Received Status: 201
    Created Custom Post. Title: Custom Title via Builder, UserID: 99
```

## Conclusion

By introducing POJOs and the Builder pattern, your automation framework is now highly resilient. If the backend team introduces a new required `authorName` field to the `/posts` endpoint, you only need to update the `PostPayloadBuilder.ts` file in one place, and all 1000 of your tests will instantly inherit the fix!

Next up, we will cover **Cookie Management** and **Session Handling**!
