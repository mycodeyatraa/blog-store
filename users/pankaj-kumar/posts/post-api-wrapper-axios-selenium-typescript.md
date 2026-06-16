---
title: Creating Data: Automating POST Requests
date: 03-Dec-2024
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [typescript, selenium, api-testing, axios, post-request]
category: Selenium TypeScript
categories: [Selenium TypeScript, API Testing Integration]
excerpt: >-
  Build Hybrid UI-API automation. Learn how to dynamically inject JSON payloads and headers into Axios to instantly create data using POST endpoints.
readTime: 5 min read
---

# Creating Data: Automating POST Requests

In the previous tutorial, we built an `ApiUtils` method to execute `GET` requests. However, reading data is only half the battle.

To write true Hybrid Tests, you often need to **create data**. 
For example: *Send an API `POST` request to create a user, then log into the UI with that user.*

Let's expand our wrapper to handle `POST` requests and JSON payloads!

---

## 1. Expanding the ApiUtils Class

Unlike a `GET` request, a `POST` request requires three things:
1. The Endpoint URL.
2. The Request Body (Payload).
3. The Headers (telling the server we are sending JSON).

Update `tests/utils/ApiUtils.ts` to include the new `post` method:

```typescript
  /**
   * Performs an HTTP POST request to create data, accepting a JSON payload.
   */
  static async post(endpoint: string, payload: any): Promise<AxiosResponse> {
    const baseUrl = ConfigManager.get("API_BASE_URL");
    const fullUrl = `${baseUrl}${endpoint}`;
    Logger.info(`[API] Sending POST request to: ${fullUrl}`);
    Logger.info(`[API] Payload: ${JSON.stringify(payload)}`);
    try {
      const response = await axios.post(fullUrl, payload, {
        headers: { "Content-Type": "application/json" }
      });
      Logger.info(`[API] Received Status: ${response.status}`);
      return response;
    } catch (error: any) {
      Logger.error(`[API] POST request failed: ${error.message}`);
      if (error.response) {
        return error.response;
      }
      throw error;
    }
  }
```

---

## 2. Writing the POST Test Script

Let's write a test that creates a new post on our mock API. We will validate that the API responds with a `201 Created` status code and returns our exact payload along with a new Database ID.

Create `tests/api_post.test.ts`:

```typescript
import { ApiUtils } from "./utils/ApiUtils";
describe("Phase 5 - API Testing: POST Requests", () => {
  it("Should successfully create a new user via POST request", async () => {
    // 1. Define the JSON payload
    const payload = {
      title: "Automation Engineer",
      body: "Testing POST APIs with Axios",
      userId: 101
    };
    // 2. Send the POST request via our Utility
    const response = await ApiUtils.post("/posts", payload);
    // 3. Validate Status (201 Created)
    expect(response.status).toBe(201);
    // 4. Validate Response Payload mirrors our request
    const responseBody = response.data;
    expect(responseBody.title).toBe("Automation Engineer");
    expect(responseBody.body).toBe("Testing POST APIs with Axios");
    // Most APIs will assign a new ID to created entities
    expect(responseBody.id).toBeDefined();
    console.log(`Successfully created Post with ID: ${responseBody.id}`);
  });
});
```

---

## 3. Test Execution Output

Run the test pointing to the QA environment:

```bash
> ENV_NAME=qa jest tests/api_post.test.ts
```

Output:

```text
2026-12-03 10:15:30 [INFO]: Loading Configuration for Environment: [QA]
2026-12-03 10:15:30 [INFO]: [API] Sending POST request to: https://jsonplaceholder.typicode.com/posts
2026-12-03 10:15:30 [INFO]: [API] Payload: {"title":"Automation Engineer","body":"Testing POST APIs with Axios","userId":101}
2026-12-03 10:15:31 [INFO]: [API] Received Status: 201
  console.log
    Successfully created Post with ID: 101
```

## Conclusion

We now have the ability to read (`GET`) and write (`POST`) data instantly against the backend!

You are now fully equipped to build **Hybrid Automation Scripts**. Need a user to exist before your UI test starts? Use `ApiUtils.post()` in the `beforeAll` block to generate the user instantly, completely bypassing the slow UI registration forms!
