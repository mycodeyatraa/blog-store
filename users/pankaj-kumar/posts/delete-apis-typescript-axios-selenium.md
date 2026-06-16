---
title: Data Cleanup: Automating DELETE Requests
date: 08-Dec-2024
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [typescript, selenium, api-testing, axios, delete, rest-api]
category: Selenium TypeScript
categories: [Selenium TypeScript, API Testing Integration]
excerpt: >-
  Complete your CRUD API framework by automating DELETE requests with Axios and handling 204 No Content status variations in TypeScript.
readTime: 4 min read
---

# Data Cleanup: Automating DELETE Requests

Now that we have covered `GET`, `POST`, `PUT`, and `PATCH`, there is only one piece missing to complete the basic CRUD lifecycle in API testing: **`DELETE`**.

A well-designed automation suite must not only generate data but also clean up after itself to avoid bloating the database or leaving stale records that cause future tests to fail. 

In this tutorial, we will implement the `DELETE` method in our API utility framework.

---

## 1. Updating ApiUtils

Let's add the final major REST action to our `tests/utils/ApiUtils.ts` wrapper class. 

The `DELETE` request is often the simplest to format because it usually doesn't require a request body, only a specific endpoint (such as an ID) and an authorization token.

```typescript
  /**
   * Performs an HTTP DELETE request to remove a resource.
   */
  static async delete(endpoint: string, token?: string): Promise<AxiosResponse> {
    const baseUrl = ConfigManager.get("API_BASE_URL");
    const fullUrl = `${baseUrl}${endpoint}`;
    Logger.info(`[API] Sending DELETE request to: ${fullUrl}`);
    const headers: Record<string, string> = {};
    if (token) {
      headers["Authorization"] = `Bearer ${token}`;
    }
    try {
      const response = await axios.delete(fullUrl, { headers });
      Logger.info(`[API] Received Status: ${response.status}`);
      return response;
    } catch (error: any) {
      Logger.error(`[API] DELETE request failed: ${error.message}`);
      if (error.response) return error.response;
      throw error;
    }
  }
```

---

## 2. Status Code Variations in DELETE

When asserting against a `DELETE` request, it is essential to understand the typical HTTP responses your backend developers might return:

1. **`200 OK`**: The resource was deleted successfully, and the server responded with a message (e.g., `{"message": "Post successfully deleted."}`).
2. **`204 No Content`**: The resource was deleted successfully, and the server explicitly returns *nothing* in the response body. This is arguably the most REST-compliant response.
3. **`404 Not Found`**: You tried to delete an ID that doesn't exist.

Our public test API (`jsonplaceholder`) responds with a `200 OK` and an empty object for `DELETE` calls.

---

## 3. Writing the DELETE Test

Create `tests/api_delete.test.ts`:

```typescript
import { ApiUtils } from "./utils/ApiUtils";
describe("Phase 6 - API Testing: DELETE Requests", () => {
  it("Should successfully delete a resource", async () => {
    // 1. Send the DELETE request targeting Post ID 1
    const response = await ApiUtils.delete("/posts/1");
    // 2. Validate Status. jsonplaceholder returns 200 OK for delete.
    // In many modern APIs, this will return a 204 No Content.
    expect([200, 204]).toContain(response.status);
    // 3. Optional: In a real system, you would try to GET the resource and verify it returns a 404
    // const getResponse = await ApiUtils.get("/posts/1");
    // expect(getResponse.status).toBe(404);
    console.log(`Successfully executed DELETE request on Post ID 1. Received Status: ${response.status}`);
  });
});
```

*(Note: Because we use `.toContain()`, our script is robust enough to pass whether the server decides to return a `200` or a `204`!)*

---

## 4. Test Execution Output

Run the test pointing to the QA environment:

```bash
> ENV_NAME=qa jest tests/api_delete.test.ts
```

Output:

```text
 PASS  tests/api_delete.test.ts
  Phase 6 - API Testing: DELETE Requests
    √ Should successfully delete a resource (206 ms)
  console.log
    [API] Sending DELETE request to: https://jsonplaceholder.typicode.com/posts/1
    [API] Received Status: 200
    Successfully executed DELETE request on Post ID 1. Received Status: 200
```

## Conclusion

We have now successfully mapped all basic CRUD operations! Your `ApiUtils` library is robust and fully functional. 

In the upcoming tutorials, we will dive into more advanced scenarios: Token Extraction (Auth APIs) and Hybrid UI+API testing!
