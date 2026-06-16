---
title: Updating Data: Automating PUT and PATCH Requests
date: 07-Dec-2024
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [typescript, selenium, api-testing, axios, put, patch, rest-api]
category: Selenium TypeScript
categories: [Selenium TypeScript, API Testing Integration]
excerpt: >-
  Understand the critical differences between full replacements and partial updates as we implement PUT and PATCH automation wrappers in TypeScript.
readTime: 4 min read
---

# Updating Data: Automating PUT and PATCH Requests

In previous API tutorials, we covered `GET` (reading) and `POST` (creating). But what happens when you need to **update** existing data?

REST APIs generally provide two different HTTP methods for modifying resources: `PUT` and `PATCH`. While they might seem similar, they have vastly different rules.

In this tutorial, we will expand our `ApiUtils` framework to handle both methods!

---

## 1. PUT vs PATCH: What is the difference?

* **`PUT` (Full Replacement):** A `PUT` request completely overwrites the target resource. You must provide the **entire object payload**, even the fields you aren't changing. If you omit a field during a `PUT` request, the backend will often delete that field or reset it to null.
* **`PATCH` (Partial Update):** A `PATCH` request only modifies the specific fields you provide. You can send a payload with just a single field (e.g., `{"title": "New Title"}`), and the backend will leave all other fields untouched.

---

## 2. Expanding ApiUtils

Update `tests/utils/ApiUtils.ts` to include our new `put` and `patch` wrappers:

```typescript
  /**
   * Performs an HTTP PUT request to replace an entire resource.
   */
  static async put(endpoint: string, payload: any, token?: string): Promise<AxiosResponse> {
    const baseUrl = ConfigManager.get("API_BASE_URL");
    const fullUrl = `${baseUrl}${endpoint}`;
    const headers: Record<string, string> = { "Content-Type": "application/json" };
    if (token) headers["Authorization"] = `Bearer ${token}`;
    try {
      return await axios.put(fullUrl, payload, { headers });
    } catch (error: any) {
      if (error.response) return error.response;
      throw error;
    }
  }
  /**
   * Performs an HTTP PATCH request to partially update a resource.
   */
  static async patch(endpoint: string, payload: any, token?: string): Promise<AxiosResponse> {
    const baseUrl = ConfigManager.get("API_BASE_URL");
    const fullUrl = `${baseUrl}${endpoint}`;
    const headers: Record<string, string> = { "Content-Type": "application/json" };
    if (token) headers["Authorization"] = `Bearer ${token}`;
    try {
      return await axios.patch(fullUrl, payload, { headers });
    } catch (error: any) {
      if (error.response) return error.response;
      throw error;
    }
  }
```

---

## 3. Writing the Update Tests

Let's write a script that demonstrates the difference between `PUT` and `PATCH`.

Create `tests/api_update.test.ts`:

```typescript
import { ApiUtils } from "./utils/ApiUtils";
describe("Phase 6 - API Testing: PUT vs PATCH Requests", () => {
  it("Should perform a PUT request to replace an entire resource", async () => {
    // 1. PUT replaces the entire object. We must provide all required fields.
    const putPayload = {
      id: 1,
      title: "Updated Title via PUT",
      body: "This completely replaced the original post.",
      userId: 1
    };
    const response = await ApiUtils.put("/posts/1", putPayload);
    // 2. Validate Status (200 OK)
    expect(response.status).toBe(200);
    // 3. Validate Response Payload
    const responseBody = response.data;
    expect(responseBody.title).toBe("Updated Title via PUT");
    expect(responseBody.body).toBe("This completely replaced the original post.");
    console.log(`Successfully replaced Post ID 1 via PUT`);
  });
  it("Should perform a PATCH request to partially update a resource", async () => {
    // 1. PATCH only updates specific fields. We don't need to provide the entire object.
    const patchPayload = {
      title: "Partially Updated Title via PATCH"
    };
    const response = await ApiUtils.patch("/posts/1", patchPayload);
    // 2. Validate Status (200 OK)
    expect(response.status).toBe(200);
    // 3. Validate Response Payload
    const responseBody = response.data;
    expect(responseBody.title).toBe("Partially Updated Title via PATCH");
    // The original body/userId remains untouched by a PATCH request
    console.log(`Successfully updated Post ID 1 via PATCH`);
  });
});
```

---

## 4. Test Execution Output

Run the test pointing to the QA environment:

```bash
> ENV_NAME=qa jest tests/api_update.test.ts
```

Output:

```text
 PASS  tests/api_update.test.ts
  Phase 6 - API Testing: PUT vs PATCH Requests
    √ Should perform a PUT request to replace an entire resource (214 ms)
    √ Should perform a PATCH request to partially update a resource (208 ms)
  console.log
    Successfully replaced Post ID 1 via PUT
  console.log
    Successfully updated Post ID 1 via PATCH
```

## Conclusion

Understanding when to use `PUT` versus `PATCH` is a hallmark of a Senior SDET. In our next tutorial, we will complete the CRUD lifecycle by covering `DELETE` requests!
