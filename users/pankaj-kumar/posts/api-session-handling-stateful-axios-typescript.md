---
title: Building a Stateful API Framework: Session Handling in TypeScript
date: 15-Dec-2024
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [typescript, selenium, api-testing, cookies, axios, session-management, stateful]
category: Selenium TypeScript
categories: [Selenium TypeScript, API Testing Integration]
excerpt: >-
  Transform stateless HTTP requests into a fully stateful, browser-like experience by encapsulating cookies into a reusable SessionManager using axios.create().
readTime: 5 min read
---

# Building a Stateful API Framework: Session Handling in TypeScript

In our previous tutorial on **Cookie Management**, we learned how to manually extract a `Set-Cookie` header and inject it into our next Axios request via the `Cookie` header.

However, manually passing the cookie into every single request is incredibly tedious. What if you have 50 API calls in a test? 

In this tutorial, we will architect a **Session Manager**. We will use `axios.create()` to generate a custom, stateful Axios instance that automatically persists cookies across your entire test lifecycle!

---

## 1. Architecting the Session Manager

We want to build a utility class that handles the login exactly once, stores the cookie, and then vends out a pre-configured Axios instance to any test that needs it.

Create a new file `tests/utils/SessionManager.ts`:

```typescript
import axios, { AxiosInstance } from "axios";
import { Logger } from "./Logger";
export class SessionManager {
  private static instance: AxiosInstance;
  private static cookies: string[] = [];
  /**
   * Initializes a stateful Axios instance by performing a login
   * and capturing the Set-Cookie headers.
   */
  public static async loginAndCreateSession(): Promise<AxiosInstance> {
    if (!this.instance) {
      Logger.info("[Session] Initializing new API Session...");
      // 1. Perform Login (using httpbin for demonstration)
      const response = await axios.get("https://httpbin.org/cookies/set/session_id/SUPER_SECRET_TOKEN", {
        maxRedirects: 0,
        validateStatus: (status) => status >= 200 && status < 400
      });
      // 2. Extract and store cookies
      const setCookieHeaders = response.headers["set-cookie"];
      if (setCookieHeaders) {
        this.cookies = setCookieHeaders.map(header => header.split(";")[0]);
        Logger.info(`[Session] Cookies captured: ${this.cookies.join("; ")}`);
      }
      // 3. Create a stateful Axios Instance
      this.instance = axios.create({
        baseURL: "https://httpbin.org",
        headers: {
          "Cookie": this.cookies.join("; ")
        }
      });
    }
    return this.instance;
  }
  /**
   * Returns the active stateful session.
   */
  public static getSession(): AxiosInstance {
    if (!this.instance) {
      throw new Error("[Session] Session not initialized! Call loginAndCreateSession() first.");
    }
    return this.instance;
  }
  /**
   * Clears the session to log out.
   */
  public static logout(): void {
    Logger.info("[Session] Logging out and destroying session...");
    this.instance = undefined as any;
    this.cookies = [];
  }
}
```

### Why use `axios.create()`?
`axios.create()` allows us to define default headers and base URLs. By injecting the `Cookie` array into the `headers` property of this custom instance, every subsequent `session.get()` or `session.post()` will automatically include the cookie!

---

## 2. Using the Stateful Session in Tests

Now, let's write our tests to consume this new `SessionManager`.

Create `tests/api_session.test.ts`:

```typescript
import { SessionManager } from "./utils/SessionManager";
describe("Phase 6 - API Testing: Session Handling", () => {
  beforeAll(async () => {
    // Initialize the session ONCE before all tests run
    await SessionManager.loginAndCreateSession();
  });
  afterAll(() => {
    // Destroy the session after tests finish
    SessionManager.logout();
  });
  it("Should make a stateful GET request using the SessionManager", async () => {
    // 1. Retrieve the pre-configured stateful Axios instance
    const session = SessionManager.getSession();
    // 2. Make a request without manually passing cookies
    // The instance already has the Cookie header injected!
    const response = await session.get("/cookies");
    // 3. Validate that the server recognizes our session
    expect(response.status).toBe(200);
    expect(response.data.cookies).toHaveProperty("session_id", "SUPER_SECRET_TOKEN");
    console.log(`Stateful Request Successful! Server Cookies:`, response.data.cookies);
  });
  it("Should maintain the session across multiple tests seamlessly", async () => {
    const session = SessionManager.getSession();
    const response = await session.get("/cookies");
    expect(response.status).toBe(200);
    expect(response.data.cookies).toHaveProperty("session_id", "SUPER_SECRET_TOKEN");
  });
});
```

Notice how clean the test is! We simply call `session.get("/cookies")` without worrying about headers, tokens, or configuration. The `SessionManager` handles everything behind the scenes.

---

## 3. Test Execution Output

Run the test:

```bash
> ENV_NAME=qa jest tests/api_session.test.ts
```

Output:

```text
 PASS  tests/api_session.test.ts
  Phase 6 - API Testing: Session Handling
    √ Should make a stateful GET request using the SessionManager (143 ms)
    √ Should maintain the session across multiple tests seamlessly (121 ms)
  console.log
    [Session] Initializing new API Session...
    [Session] Cookies captured: session_id=SUPER_SECRET_TOKEN
  console.log
    Stateful Request Successful! Server Cookies: { session_id: 'SUPER_SECRET_TOKEN' }
  console.log
    [Session] Logging out and destroying session...
```

## Conclusion

By encapsulating cookie extraction and header injection into a `SessionManager` utilizing `axios.create()`, we transformed stateless HTTP requests into a fully stateful, browser-like experience. 

This design pattern is essential for writing scalable, maintainable enterprise API automation frameworks! In our next and final API tutorial, we will explore **Multi-User Testing**, where we instantiate multiple parallel sessions for complex role-based testing!
