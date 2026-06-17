---
title: Automating API Sessions: Managing HTTP Cookies in Axios
date: 14-Dec-2024
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [typescript, selenium, api-testing, cookies, axios, session-management]
category: Selenium TypeScript
categories: [Selenium TypeScript, API Testing Integration]
excerpt: >-
  Bypass UI logins by learning how to manually extract Set-Cookie headers and inject them into subsequent Axios API requests using TypeScript.
readTime: 4 min read
---

# Automating API Sessions: Managing HTTP Cookies in Axios

While JWT Bearer tokens are the most common form of API authentication, many legacy systems and web applications still rely heavily on **HTTP Cookies** for session management. 

When you log in via the UI, the backend server responds with a `Set-Cookie` header. Your browser automatically saves this cookie and injects it into every subsequent request via the `Cookie` header.

However, when automating API tests with Axios in Node.js, **you do not have a browser**. Axios will ignore the `Set-Cookie` header by default!

In this tutorial, we will learn how to manually extract and inject cookies to maintain API sessions.

---

## 1. Extracting the Set-Cookie Header

To maintain a session, we must intercept the initial response from the backend and save the cookie string. We will use `httpbin.org` as our mock server because it allows us to test cookie behaviors securely.

Create `tests/api_cookies.test.ts`:

```typescript
import axios from "axios";
describe("Phase 6 - API Testing: Cookie Management", () => {
  let sessionCookie: string;
  it("Should extract the Set-Cookie header from a response", async () => {
    // 1. Hit an endpoint that sets a cookie
    // httpbin.org/cookies/set sets a cookie and redirects (302).
    const response = await axios.get("https://httpbin.org/cookies/set/session_id/12345ABCDE", {
      maxRedirects: 0, // Prevent redirect so we can read the raw Set-Cookie header from the 302
      validateStatus: (status) => status >= 200 && status < 400
    });
    // 2. Extract the Set-Cookie header array
    const setCookieHeaders = response.headers["set-cookie"];
    expect(setCookieHeaders).toBeDefined();
    // 3. Parse out the specific cookie we want
    if (setCookieHeaders) {
      // The header looks like: "session_id=12345ABCDE; Path=/"
      sessionCookie = setCookieHeaders[0].split(";")[0]; 
    }
    console.log(`Extracted Session Cookie: ${sessionCookie}`);
    expect(sessionCookie).toContain("session_id=12345ABCDE");
  });
  // ... next step
});
```

**Key Concept**: We pass `maxRedirects: 0` because HTTP redirects often drop the original `Set-Cookie` headers if you let Axios automatically follow them to the final 200 OK page.

---

## 2. Injecting the Cookie

Now that we have the cookie string stored in our `sessionCookie` variable, we need to pass it back to the server in our subsequent requests.

Append the following test to your file:

```typescript
  it("Should inject the extracted cookie into subsequent requests", async () => {
    // 1. Inject the cookie into the 'Cookie' header
    const response = await axios.get("https://httpbin.org/cookies", {
      headers: {
        "Cookie": sessionCookie // Bypassing Axios defaults!
      }
    });
    // 2. Validate that the server received our cookie
    expect(response.status).toBe(200);
    expect(response.data.cookies).toHaveProperty("session_id", "12345ABCDE");
    console.log(`Successfully passed cookie to server. Server sees: ${JSON.stringify(response.data.cookies)}`);
  });
```

By explicitly setting the `Cookie` header inside the Axios config object, we are effectively spoofing a browser session! The server receives the request, sees the valid session ID, and grants us access.

---

## 3. Test Execution Output

Run the test:

```bash
> ENV_NAME=qa jest tests/api_cookies.test.ts
```

Output:

```text
 PASS  tests/api_cookies.test.ts
  Phase 6 - API Testing: Cookie Management
    √ Should extract the Set-Cookie header from a response (321 ms)
    √ Should inject the extracted cookie into subsequent requests (298 ms)
  console.log
    Extracted Session Cookie: session_id=12345ABCDE
  console.log
    Successfully passed cookie to server. Server sees: {"session_id":"12345ABCDE"}
```

## Conclusion

Understanding how to read and write HTTP headers is the foundation of advanced backend automation. By manually extracting `Set-Cookie` and injecting `Cookie` headers, you can bypass complex UI login screens and execute secure endpoints entirely within your API framework.

In our very next tutorial, we will explore **Session Handling**, combining everything we've learned to build persistent, stateful automation workflows!
