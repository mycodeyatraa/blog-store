---
title: Building an API Wrapper: Automating GET Requests
date: 02-Dec-2024
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [typescript, selenium, api-testing, axios, environment-variables]
category: Selenium TypeScript
categories: [Selenium TypeScript, API Testing Integration]
excerpt: >-
  Integrate API automation seamlessly into your UI framework by building an ApiUtils class that dynamically extracts URLs from your .env configurations.
readTime: 5 min read
---

# Building an API Wrapper: Automating GET Requests

In our previous tutorial, we performed a basic Axios `GET` request using a hardcoded URL.

However, in an Enterprise Framework, Backend APIs are deployed to different environments just like Frontends. You might have:
* QA API: `https://qa-api.mycodeyatra.com`
* UAT API: `https://uat-api.mycodeyatra.com`

If you hardcode these URLs in your tests, you'll have to rewrite them every time you execute against a new environment. 

To solve this, we will use the `ConfigManager` we built in Phase 4 to construct a centralized **ApiUtils** wrapper class!

---

## 1. Updating the Environment Config

First, let's add a new key to our `.env.qa` file to store the base URL of our backend API.

Update `.env.qa`:

```ini
BASE_URL=https://practice.mycodeyatra.com/#/form-practice
ADMIN_USERNAME=qa_admin
ADMIN_PASSWORD=qa_password_123
API_TIMEOUT=15000
API_BASE_URL=https://jsonplaceholder.typicode.com
```

---

## 2. Building the ApiUtils Class

We will create a static helper class that automatically prepends the `API_BASE_URL` to any endpoint we pass in.

We will also add a `try-catch` block. By default, Axios throws a fatal JavaScript error if an API returns a `404` or `500` status. We want to catch that error and return the response object so our Jest tests can smoothly assert the failure!

Create `tests/utils/ApiUtils.ts`:

```typescript
import axios, { AxiosResponse } from "axios";
import { ConfigManager } from "./ConfigManager";
import { Logger } from "./Logger";
export class ApiUtils {
  /**
   * Performs a robust HTTP GET request using the configured API_BASE_URL.
   */
  static async get(endpoint: string): Promise<AxiosResponse> {
    // Dynamically fetch the URL from .env
    const baseUrl = ConfigManager.get("API_BASE_URL");
    const fullUrl = `${baseUrl}${endpoint}`;
    Logger.info(`[API] Sending GET request to: ${fullUrl}`);
    try {
      const response = await axios.get(fullUrl);
      Logger.info(`[API] Received Status: ${response.status}`);
      return response;
    } catch (error: any) {
      Logger.error(`[API] GET request failed: ${error.message}`);
      // Return the error response so Jest can assert it without crashing!
      if (error.response) {
        return error.response; 
      }
      throw error;
    }
  }
}
```

---

## 3. Writing the Dynamic API Test

Now our tests are completely environment-agnostic. We simply pass the endpoint suffix (e.g., `/users/2`), and the Utility Framework handles the rest!

Create `tests/api_get.test.ts`:

```typescript
import { ApiUtils } from "./utils/ApiUtils";
describe("Phase 5 - API Testing: Framework Integration", () => {
  it("Should perform a dynamic GET request using the ApiUtils Framework", async () => {
    // 1. We no longer hardcode the base URL!
    const response = await ApiUtils.get("/users/2");
    // 2. Validate Response
    expect(response.status).toBe(200);
    const user = response.data;
    expect(user.id).toBe(2);
    expect(user.username).toBe("Antonette");
    expect(user.email).toBeDefined();
    console.log(`Validated dynamic GET request for user: ${user.username}`);
  });
  it("Should return a 404 response natively without crashing Jest", async () => {
    // 3. Testing negative scenarios is now incredibly clean
    const response = await ApiUtils.get("/users/9999");
    expect(response.status).toBe(404);
  });
});
```

---

## 4. Test Execution Output

When we run the tests:

```bash
> ENV_NAME=qa jest tests/api_get.test.ts
```

Our Winston logger automatically captures the backend activity:

```text
2026-12-02 10:15:30 [INFO]: Loading Configuration for Environment: [QA]
2026-12-02 10:15:30 [INFO]: [API] Sending GET request to: https://jsonplaceholder.typicode.com/users/2
2026-12-02 10:15:30 [INFO]: [API] Received Status: 200
  console.log
    Validated dynamic GET request for user: Antonette
2026-12-02 10:15:30 [INFO]: [API] Sending GET request to: https://jsonplaceholder.typicode.com/users/9999
2026-12-02 10:15:31 [ERROR]: [API] GET request failed: Request failed with status code 404
```

## Conclusion

By abstracting our Axios logic into an `ApiUtils` class, we have unified our API framework with our UI framework. Both now share the same Environment Configs and the same Winston Logging system!

In our next tutorial, we will expand `ApiUtils` to handle **POST APIs**, allowing us to physically inject JSON payloads into our backend to create data!
