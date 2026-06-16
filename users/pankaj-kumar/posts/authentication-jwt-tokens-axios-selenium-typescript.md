---
title: Mastering API Authentication: JWT Tokens and Bearer Headers
date: 09-Dec-2024
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [typescript, selenium, api-testing, axios, authentication, jwt, bearer-token]
category: Selenium TypeScript
categories: [Selenium TypeScript, API Testing Integration]
excerpt: >-
  Learn to bypass security by dynamically extracting JWT authentication tokens from APIs and injecting them into your Axios requests using TypeScript.
readTime: 5 min read
---

# Mastering API Authentication: JWT Tokens and Bearer Headers

Almost every modern enterprise application is secured by authentication. Whether it is OAuth 2.0, SAML, or JWT (JSON Web Tokens), you cannot test private APIs unless you prove *who* you are.

In this tutorial, we will automate the extraction of a JWT token from a Login API and inject it into our subsequent requests via Bearer Headers.

---

## 1. Creating the Authentication Utility

Because `jsonplaceholder` does not support real authentication, we will use a different free mock API for this specific lesson: `https://dummyjson.com`. 

Create a new file `tests/utils/AuthUtils.ts`:

```typescript
import axios from "axios";
import { Logger } from "./Logger";
export class AuthUtils {
  /**
   * Retrieves a JWT Authentication Token by logging in via API.
   * We use dummyjson.com for this demonstration.
   */
  static async getAuthToken(username: string = "emilys", password: string = "emilyspass"): Promise<string> {
    Logger.info(`[Auth] Attempting to retrieve JWT token for user: ${username}`);
    try {
      const response = await axios.post("https://dummyjson.com/auth/login", {
        username: username,
        password: password
      }, {
        headers: { "Content-Type": "application/json" }
      });
      if (response.status === 200 && response.data.token) {
        Logger.info(`[Auth] Token successfully retrieved!`);
        return response.data.token;
      } else {
        throw new Error(`[Auth] Failed to retrieve token. Status: ${response.status}`);
      }
    } catch (error: any) {
      Logger.error(`[Auth] Login API request failed: ${error.message}`);
      throw error;
    }
  }
}
```

This utility performs a `POST` request to the `/auth/login` endpoint with a username and password. The server responds with a JSON object containing the `token`. We parse and return it!

---

## 2. Writing the Authenticated Test Suite

Now, we will use this utility inside a test. We want to fetch the token exactly **once** before all tests run, and reuse it across multiple assertions.

Create `tests/api_auth.test.ts`:

```typescript
import { AuthUtils } from "./utils/AuthUtils";
import axios from "axios";
describe("Phase 6 - API Testing: Authentication APIs", () => {
  let jwtToken: string;
  beforeAll(async () => {
    // 1. Authenticate before tests run to get the JWT
    jwtToken = await AuthUtils.getAuthToken();
    console.log(`Generated JWT Token: ${jwtToken.substring(0, 15)}...`);
  });
  it("Should access a protected route using the JWT token", async () => {
    // 2. We use dummyjson.com's /auth/me endpoint to verify the token
    const response = await axios.get("https://dummyjson.com/auth/me", {
      headers: {
        "Authorization": `Bearer ${jwtToken}` // Injecting the Bearer Token
      }
    });
    // 3. Validate the Response
    expect(response.status).toBe(200);
    expect(response.data.username).toBe("emilys");
    console.log(`Successfully validated JWT Token for user: ${response.data.username}`);
  });
  it("Should fail when accessing a protected route without a token", async () => {
    try {
      await axios.get("https://dummyjson.com/auth/me");
    } catch (error: any) {
      // 4. Validate that a 401 Unauthorized or 403 Forbidden is returned
      expect([401, 403]).toContain(error.response.status);
      console.log(`Successfully blocked unauthorized access. Status: ${error.response.status}`);
    }
  });
});
```

### Key Takeaways
1. **`beforeAll` Hook**: Notice how we fetch the token inside the `beforeAll` block. We don't want to log in 10 times if we have 10 tests; that slows down execution.
2. **Bearer Token Injection**: In the first test, we pass a `headers` object containing `Authorization: Bearer <TOKEN>`.
3. **Negative Testing**: The second test intentionally omits the token and expects the server to reject the request with a `401 Unauthorized` or `403 Forbidden` status.

---

## 3. Test Execution Output

Run the test pointing to the QA environment:

```bash
> ENV_NAME=qa jest tests/api_auth.test.ts
```

Output:

```text
 PASS  tests/api_auth.test.ts
  Phase 6 - API Testing: Authentication APIs
    √ Should access a protected route using the JWT token (241 ms)
    √ Should fail when accessing a protected route without a token (198 ms)
  console.log
    [Auth] Attempting to retrieve JWT token for user: emilys
    [Auth] Token successfully retrieved!
    Generated JWT Token: eyJhbGciOiJIUzI...
  console.log
    Successfully validated JWT Token for user: emilys
  console.log
    Successfully blocked unauthorized access. Status: 401
```

## Conclusion

Automating API authentication dynamically unlocks your ability to test backend services securely. This is also the exact technique used when building Hybrid UI-API testing frameworks, which we will explore next!
