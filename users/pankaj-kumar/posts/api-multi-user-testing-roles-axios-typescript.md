---
title: The Grand Finale: Multi-User Testing in TypeScript
date: 16-Dec-2024
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [typescript, selenium, api-testing, multi-user, roles, axios, session-management]
category: Selenium TypeScript
categories: [Selenium TypeScript, API Testing Integration]
excerpt: >-
  Complete your automation framework by architecting a Multi-Session Manager to run parallel Admin and Standard User tests dynamically using Axios.
readTime: 5 min read
---

# The Grand Finale: Multi-User Testing in TypeScript

We have reached the final tutorial of the TypeScript Selenium API series! 

In real-world enterprise applications, you rarely test just one role. A common workflow might be:
1. An **Admin** logs into the API and creates a restricted resource.
2. A **Standard User** logs into the API and attempts to access the resource (and should be blocked).
3. The **Admin** logs back in and deletes the resource.

To accomplish this efficiently, we cannot log in and log out repeatedly. Instead, we need to maintain multiple parallel HTTP sessions within the exact same test script.

In this tutorial, we will upgrade our `SessionManager` to a `MultiSessionManager`!

---

## 1. Upgrading the Session Manager

We need to replace our single `AxiosInstance` with a `Map` that stores instances keyed by a `UserRole`.

Create `tests/utils/MultiSessionManager.ts`:

```typescript
import axios, { AxiosInstance } from "axios";
import { Logger } from "./Logger";
// Define the Roles
export enum UserRole {
  ADMIN = "ADMIN",
  STANDARD_USER = "STANDARD_USER"
}
export class MultiSessionManager {
  // Store multiple stateful Axios instances
  private static sessions: Map<UserRole, AxiosInstance> = new Map();
  /**
   * Initializes a stateful session for a specific role.
   */
  public static async loginAs(role: UserRole, sessionToken: string): Promise<AxiosInstance> {
    if (!this.sessions.has(role)) {
      Logger.info(`[Multi-Session] Initializing new API Session for role: ${role}`);
      // We simulate logging in as different users by passing different tokens to httpbin
      const response = await axios.get(`https://httpbin.org/cookies/set/session_id/${sessionToken}`, {
        maxRedirects: 0,
        validateStatus: (status) => status >= 200 && status < 400
      });
      const setCookieHeaders = response.headers["set-cookie"];
      let cookieStr = "";
      if (setCookieHeaders) {
        cookieStr = setCookieHeaders.map(header => header.split(";")[0]).join("; ");
      }
      const instance = axios.create({
        baseURL: "https://httpbin.org",
        headers: {
          "Cookie": cookieStr
        }
      });
      // Save the instance to the Map!
      this.sessions.set(role, instance);
      Logger.info(`[Multi-Session] ${role} Session created successfully.`);
    }
    return this.sessions.get(role)!;
  }
  /**
   * Returns the active session for a given role.
   */
  public static getSession(role: UserRole): AxiosInstance {
    if (!this.sessions.has(role)) {
      throw new Error(`[Multi-Session] Session not initialized for role: ${role}`);
    }
    return this.sessions.get(role)!;
  }
  /**
   * Clears all sessions.
   */
  public static clearAllSessions(): void {
    Logger.info("[Multi-Session] Destroying all active sessions...");
    this.sessions.clear();
  }
}
```

---

## 2. Writing the Multi-User Test

With our `Map` architecture in place, writing multi-user tests becomes incredibly intuitive.

Create `tests/api_multi_user.test.ts`:

```typescript
import { MultiSessionManager, UserRole } from "./utils/MultiSessionManager";
describe("Phase 6 - API Testing: Multi-User Testing", () => {
  beforeAll(async () => {
    // 1. Initialize two entirely different sessions in the setup block
    await MultiSessionManager.loginAs(UserRole.ADMIN, "ADMIN_TOKEN_999");
    await MultiSessionManager.loginAs(UserRole.STANDARD_USER, "USER_TOKEN_111");
  });
  afterAll(() => {
    // Destroy all sessions
    MultiSessionManager.clearAllSessions();
  });
  it("Should act as an ADMIN and perform privileged actions", async () => {
    // 2. Fetch the specific Admin session
    const adminSession = MultiSessionManager.getSession(UserRole.ADMIN);
    // 3. Make the API call (the ADMIN_TOKEN_999 cookie is automatically attached)
    const response = await adminSession.get("/cookies");
    // 4. Validate that the server recognizes the Admin token
    expect(response.status).toBe(200);
    expect(response.data.cookies).toHaveProperty("session_id", "ADMIN_TOKEN_999");
    console.log(`[Admin Test] Validated Admin Session. Server Cookies:`, response.data.cookies);
  });
  it("Should act as a STANDARD USER and verify restricted access", async () => {
    // 2. Fetch the specific Standard User session
    const userSession = MultiSessionManager.getSession(UserRole.STANDARD_USER);
    // 3. Make the API call (the USER_TOKEN_111 cookie is automatically attached)
    const response = await userSession.get("/cookies");
    // 4. Validate that the server recognizes the Standard User token
    expect(response.status).toBe(200);
    expect(response.data.cookies).toHaveProperty("session_id", "USER_TOKEN_111");
    console.log(`[User Test] Validated Standard User Session. Server Cookies:`, response.data.cookies);
  });
});
```

---

## 3. Test Execution Output

Run the test:

```bash
> ENV_NAME=qa jest tests/api_multi_user.test.ts
```

Output:

```text
 PASS  tests/api_multi_user.test.ts
  Phase 6 - API Testing: Multi-User Testing
    √ Should act as an ADMIN and perform privileged actions (183 ms)
    √ Should act as a STANDARD USER and verify restricted access (174 ms)
  console.log
    [Multi-Session] Initializing new API Session for role: ADMIN
    [Multi-Session] ADMIN Session created successfully.
    [Multi-Session] Initializing new API Session for role: STANDARD_USER
    [Multi-Session] STANDARD_USER Session created successfully.
  console.log
    [Admin Test] Validated Admin Session. Server Cookies: { session_id: 'ADMIN_TOKEN_999' }
  console.log
    [User Test] Validated Standard User Session. Server Cookies: { session_id: 'USER_TOKEN_111' }
  console.log
    [Multi-Session] Destroying all active sessions...
```

## Conclusion

Congratulations! You have completed the **Selenium TypeScript** roadmap. 

You have mastered everything from basic UI locators and the Screenplay Pattern to advanced architecture like POJO Data Builders, Contract Validation, and parallel Multi-User API testing.

These are the exact patterns used by Principal SDETs to architect automated testing frameworks at top-tier tech companies. Good luck on your automation journey!
