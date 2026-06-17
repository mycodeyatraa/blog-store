---
title: Automating Web Application Security: Validating HTTP Security Headers
date: 21-Dec-2024
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [typescript, selenium, security, http-headers, hsts, clickjacking]
category: Selenium TypeScript
categories: [Selenium TypeScript, Security Testing Automation]
excerpt: >-
  Step into the world of Security Testing by automating the validation of critical HTTP security headers like HSTS and X-Frame-Options using TypeScript.
readTime: 4 min read
---

# Automating Web Application Security: Validating HTTP Security Headers

Welcome to **Phase 7: Security Testing**! 

Quality Assurance is no longer just about clicking buttons and verifying business logic. In the modern CI/CD pipeline, QA Engineers are increasingly responsible for verifying the security posture of an application *before* it reaches the dedicated Security/Pen-testing teams.

The easiest and most critical first step in security automation is validating **HTTP Security Headers**. 

Security headers instruct the user's web browser on how to behave securely. If your backend developers forget to include them, your entire application is vulnerable to attacks like Clickjacking, Cross-Site Scripting (XSS), and MIME-sniffing.

In this tutorial, we will automate the validation of the "Big Three" security headers using TypeScript and Axios.

---

## 1. The "Big Three" Security Headers

1. **`Strict-Transport-Security` (HSTS)**: Forces the browser to strictly use HTTPS. It prevents Man-in-the-Middle (MitM) attacks where traffic is downgraded to unencrypted HTTP.
2. **`X-Frame-Options`**: Prevents **Clickjacking**. It tells the browser that your website is not allowed to be rendered inside an `<iframe>` on another hacker's domain.
3. **`X-Content-Type-Options`**: Prevents **MIME-sniffing**. If a hacker uploads a malicious `.js` file but names it `image.jpg`, this header stops the browser from analyzing the contents and executing the disguised JavaScript.

---

## 2. Writing the Security Header Test

We don't need Selenium WebDriver for this test. Since we are simply checking the backend's response configuration, we will use our REST API automation skills.

Create a new file `tests/security_headers.test.ts`:

```typescript
import axios from "axios";
describe("Phase 7 - Security Testing: HTTP Security Headers", () => {
  const TARGET_URL = "https://practice.mycodeyatra.com/";
  let responseHeaders: any;
  beforeAll(async () => {
    // We make a GET request to fetch the initial headers
    const response = await axios.get(TARGET_URL);
    responseHeaders = response.headers;
    console.log("[Security] Fetched HTTP Headers from target application.");
  });
  it("Should implement Strict-Transport-Security (HSTS) to force HTTPS", () => {
    // Headers are converted to lowercase in Node/Axios
    const hsts = responseHeaders["strict-transport-security"];
    // Validate the header exists
    expect(hsts).toBeDefined();
    // Validate it includes max-age
    expect(hsts).toMatch(/max-age=\d+/);
    console.log(`[Security] Validated HSTS Header: ${hsts}`);
  });
  it("Should implement X-Frame-Options to prevent Clickjacking", () => {
    const xFrameOptions = responseHeaders["x-frame-options"];
    expect(xFrameOptions).toBeDefined();
    // Usually set to DENY or SAMEORIGIN
    expect(["DENY", "SAMEORIGIN"]).toContain(xFrameOptions.toUpperCase());
    console.log(`[Security] Validated X-Frame-Options Header: ${xFrameOptions}`);
  });
  it("Should implement X-Content-Type-Options to prevent MIME-sniffing", () => {
    const xContentTypeOptions = responseHeaders["x-content-type-options"];
    expect(xContentTypeOptions).toBeDefined();
    expect(xContentTypeOptions.toLowerCase()).toBe("nosniff");
    console.log(`[Security] Validated X-Content-Type-Options Header: ${xContentTypeOptions}`);
  });
});
```

---

## 3. Test Execution Output

Run the security test:

```bash
> ENV_NAME=qa jest tests/security_headers.test.ts
```

Output:

```text
 PASS  tests/security_headers.test.ts
  Phase 7 - Security Testing: HTTP Security Headers
    √ Should implement Strict-Transport-Security (HSTS) to force HTTPS (2 ms)
    √ Should implement X-Frame-Options to prevent Clickjacking (1 ms)
    √ Should implement X-Content-Type-Options to prevent MIME-sniffing (1 ms)
  console.log
    [Security] Fetched HTTP Headers from target application.
    [Security] Validated HSTS Header: max-age=31536000; includeSubDomains
    [Security] Validated X-Frame-Options Header: SAMEORIGIN
    [Security] Validated X-Content-Type-Options Header: nosniff
```

## Conclusion

By writing a simple, sub-second API test, you have proactively verified that your application is hardened against three major security vulnerabilities. This test should run on every single Pull Request before code merges to production!

In our next tutorial, we will tackle the most complex and powerful security header of them all: the **Content Security Policy (CSP)**.
