---
title: Drastically Reduce Execution Time: Disk-Based Token Management
date: 19-Dec-2024
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [typescript, selenium, api-testing, jwt, token-caching, performance, disk-cache]
category: Selenium TypeScript
categories: [Selenium TypeScript, API Testing Integration]
excerpt: >-
  Optimize your enterprise automation suite by learning how to cache API JWT tokens to the file system to avoid rate limiting and speed up parallel execution.
readTime: 4 min read
---

# Drastically Reduce Execution Time: Disk-Based Token Management

When testing an enterprise application, you might have hundreds of API tests or UI tests running in parallel. If every single test logs in to fetch an authentication token, you create a massive bottleneck on the authentication server. 

Worse, some IDPs implement rate limiting. If you request 50 tokens in 2 seconds, your IP will be temporarily blocked.

In this tutorial, we will learn how to implement **Disk-Based Token Caching**. Instead of calling the Login API for every test, we will fetch the token *once*, save it to a `.json` file on disk, and instruct all subsequent tests to read from the file.

---

## 1. Creating the Token Manager

We will use Node's native `fs` (File System) module to read and write our authentication token to a hidden `.auth` directory.

Create `tests/utils/TokenManager.ts`:

```typescript
import fs from "fs";
import path from "path";
import { AuthUtils } from "./AuthUtils";
import { Logger } from "./Logger";
export class TokenManager {
  private static tokenFilePath = path.resolve(__dirname, "../../.auth/token.json");
  /**
   * Retrieves a valid token from disk. If no token exists, or if the current
   * token is expired, it will log in to fetch a new one and cache it.
   */
  public static async getValidToken(): Promise<string> {
    // 1. Check if token file exists
    if (fs.existsSync(this.tokenFilePath)) {
      const fileData = fs.readFileSync(this.tokenFilePath, "utf-8");
      const parsedData = JSON.parse(fileData);
      // 2. Check if token is still valid (e.g., expires in the future)
      const now = new Date().getTime();
      if (parsedData.expiresAt > now) {
        Logger.info("[TokenManager] Found valid cached token. Skipping login API call.");
        return parsedData.token;
      } else {
        Logger.info("[TokenManager] Cached token is expired. Will fetch a new one.");
      }
    } else {
      Logger.info("[TokenManager] No cached token found. Will fetch a new one.");
    }
    // 3. Fetch new token via API (Requires an actual login call)
    const newToken = await AuthUtils.getAuthToken();
    // 4. Cache the new token to disk with an expiration time (e.g. 1 hour)
    const expiresAt = new Date().getTime() + (60 * 60 * 1000); 
    const cacheData = {
      token: newToken,
      expiresAt: expiresAt
    };
    // Ensure the .auth directory exists
    const dir = path.dirname(this.tokenFilePath);
    if (!fs.existsSync(dir)) {
      fs.mkdirSync(dir, { recursive: true });
    }
    // Write token to disk
    fs.writeFileSync(this.tokenFilePath, JSON.stringify(cacheData, null, 2));
    Logger.info("[TokenManager] New token cached successfully to disk.");
    return newToken;
  }
}
```

### Why cache to disk and not Memory?
If you are running tests using a test runner like Jest with workers, each test file runs in its own isolated Node process. Memory (RAM) variables are not shared between worker processes! 

Writing the token to the file system ensures that **all parallel workers** can read the exact same authentication token.

---

## 2. Testing the Caching Mechanism

Let's write a test that verifies our caching logic works.

Create `tests/token_management.test.ts`:

```typescript
import { TokenManager } from "./utils/TokenManager";
import { AuthUtils } from "./utils/AuthUtils";
describe("Phase 6 - Advanced Authentication: Token Management", () => {
  it("Should fetch a fresh token if the cache is empty", async () => {
    // Spying on the AuthUtils to track API calls
    const spy = jest.spyOn(AuthUtils, "getAuthToken");
    const token = await TokenManager.getValidToken();
    expect(token).toBeDefined();
    // Since there was no token on disk initially, the login API is called.
    expect(spy).toHaveBeenCalledTimes(1);
    console.log(`[Test 1] Received Token: ${token}`);
    spy.mockRestore();
  });
  it("Should reuse the cached token on subsequent calls", async () => {
    const spy = jest.spyOn(AuthUtils, "getAuthToken");
    // This second call should read from disk instead of hitting the login endpoint
    const token = await TokenManager.getValidToken();
    expect(token).toBeDefined();
    // The spy should NOT be called because we returned the cached token early!
    expect(spy).not.toHaveBeenCalled();
    console.log(`[Test 2] Reused Cached Token: ${token}`);
    spy.mockRestore();
  });
});
```

---

## 3. Test Execution Output

Run the script:

```bash
> ENV_NAME=qa jest tests/token_management.test.ts
```

Output:

```text
 PASS  tests/token_management.test.ts
  Phase 6 - Advanced Authentication: Token Management
    √ Should fetch a fresh token if the cache is empty (1014 ms)
    √ Should reuse the cached token on subsequent calls (4 ms)
  console.log
    [TokenManager] No cached token found. Will fetch a new one.
    [TokenManager] New token cached successfully to disk.
    [Test 1] Received Token: eyJhbGci...
  console.log
    [TokenManager] Found valid cached token. Skipping login API call.
    [Test 2] Reused Cached Token: eyJhbGci...
```

Notice the execution times:
* Test 1 (API Login) took **1014 ms**.
* Test 2 (Disk Cache) took **4 ms**.

If you have 500 tests, caching your token saves you **8.5 minutes** of execution time on the login phase alone!

## Conclusion

Disk-caching your JWT tokens or Session Cookies is a critical optimization technique for mature automation suites. 

In our next topic, we will discuss how to combine our API session logic with our Selenium WebDriver instance to launch **Authenticated Browser Sessions**, instantly bypassing UI login screens!
