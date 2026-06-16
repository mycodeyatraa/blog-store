---
title: Breaking the Gates: API Authentication with Tokens
date: 04-Dec-2024
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [typescript, selenium, api-testing, axios, authentication, jwt]
category: Selenium TypeScript
categories: [Selenium TypeScript, API Testing Integration]
excerpt: >-
  You can't test a secure backend without proving who you are. Learn how to extract and inject Bearer Tokens into Axios request headers.
readTime: 4 min read
---

# Breaking the Gates: API Authentication with Tokens

So far, we have been communicating with public APIs. However, in the real world, almost every backend endpoint is secured behind an Authentication wall. 

If you try to hit a secure API without proving who you are, you will be rejected with a `401 Unauthorized` status.

In modern architecture, you prove your identity by logging in and receiving a **JWT (JSON Web Token)**. You must then attach this token to the `Headers` of every subsequent API request!

---

## 1. Updating ApiUtils to Accept Tokens

We must update our `GET` and `POST` wrappers to accept an optional `token` string. If the token is provided, we will automatically inject it into the `Authorization` header as a `Bearer` token.

Update `tests/utils/ApiUtils.ts`:

```typescript
  static async get(endpoint: string, token?: string): Promise<AxiosResponse> {
    const baseUrl = ConfigManager.get("API_BASE_URL");
    const fullUrl = `${baseUrl}${endpoint}`;
    // Create our Headers object
    const headers: Record<string, string> = {};
    if (token) {
      // Inject the Bearer Token!
      headers["Authorization"] = `Bearer ${token}`;
      Logger.info(`[API] Injected Bearer Token into headers`);
    }
    try {
      const response = await axios.get(fullUrl, { headers });
      return response;
    } catch (error: any) {
       // ... existing error handling
    }
  }
```
*(Do the same for the `post` method!)*

---

## 2. Automating the Secure Request

Let's write a script that tests our ability to transmit a Bearer Token.

Since `jsonplaceholder` doesn't support secure endpoints, we will use `httpbin.org/bearer`, a public tool designed specifically to test if tokens are being transmitted correctly.

Create `tests/api_auth.test.ts`:

```typescript
import axios from "axios";
describe("Phase 5 - API Testing: Authentication Tokens", () => {
  it("Should successfully inject a Bearer token into a secure request", async () => {
    // 1. In a real scenario, we would hit a login endpoint to fetch a dynamic token:
    // const loginResponse = await ApiUtils.post("/login", { user: "admin", pass: "password" });
    // const token = loginResponse.data.token;
    // For this demonstration, we will mock a JWT token
    const mockJwtToken = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.MockPayload.Signature";
    // 2. We use httpbin.org to verify our headers were attached properly.
    const response = await axios.get("https://httpbin.org/bearer", {
      headers: { "Authorization": `Bearer ${mockJwtToken}` }
    });
    expect(response.status).toBe(200);
    // 3. Verify the token was successfully transmitted
    const responseBody = response.data;
    expect(responseBody.authenticated).toBe(true);
    expect(responseBody.token).toBe(mockJwtToken);
    console.log("Successfully authenticated with secure endpoint!");
  });
});
```

---

## 3. Test Execution Output

```bash
> jest tests/api_auth.test.ts
```

Output:

```text
 PASS  tests/api_auth.test.ts
  Phase 5 - API Testing: Authentication Tokens
    √ Should successfully inject a Bearer token into a secure request (193 ms)
  console.log
    Successfully authenticated with secure endpoint!
```

## Conclusion

By dynamically extracting the JWT from a `/login` endpoint and passing it into our `ApiUtils` wrappers, we can completely automate complex Backend journeys without ever opening a browser.

We have officially mastered the core mechanics of Axios! 

In our final two API tutorials, we will explore **Data Parsing** and **Contract Validations** to ensure the backend is returning the exact schema we expect!
