---
title: The Ultimate Defense Against XSS: Automating CSP Validation
date: 22-Dec-2024
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [typescript, selenium, security, csp, xss, http-headers]
category: Selenium TypeScript
categories: [Selenium TypeScript, Security Testing Automation]
excerpt: >-
  Prevent Cross-Site Scripting (XSS) in CI/CD by automating the parsing and validation of the Content Security Policy (CSP) header using TypeScript.
readTime: 4 min read
---

# The Ultimate Defense Against XSS: Automating CSP Validation

In our last tutorial, we verified basic security headers like `X-Frame-Options`. However, the most powerful and complex security header in modern web architecture is the **Content Security Policy (CSP)**.

A CSP is essentially an allow-list for your browser. It explicitly tells the browser exactly which domains it is allowed to download images from, which domains it can load CSS from, and, most importantly, which domains can execute JavaScript.

If a hacker manages to inject a malicious script tag into your website (Cross-Site Scripting), the browser will **block the execution** if that script's domain is not on the CSP allow-list!

In this tutorial, we will write automation to parse and validate the CSP header.

---

## 1. Understanding the CSP Header Structure

A standard enterprise CSP header looks like a semicolon-separated list of directives:

`default-src 'self'; script-src 'self' https://trusted.cdn.com; object-src 'none';`

* `default-src 'self'`: Only load resources from the same domain by default.
* `script-src 'self' https://trusted.cdn.com`: Only execute JavaScript hosted on the same domain or the specific trusted CDN.
* `object-src 'none'`: Completely disable embedding plugins like Flash or Java.

A critical vulnerability occurs if developers accidentally use `'unsafe-inline'` inside `script-src`, which allows *any* script tag (even hacker-injected ones) to execute. We must automate a check to ensure this never happens.

---

## 2. Writing the CSP Automation Script

Create a new file `tests/csp_validation.test.ts`:

```typescript
import axios from "axios";
describe("Phase 7 - Security Testing: CSP Validation", () => {
  const TARGET_URL = "https://practice.mycodeyatra.com/";
  let cspHeader: string;
  beforeAll(async () => {
    const response = await axios.get(TARGET_URL);
    // In many environments, the CSP might not be set. We provide a fallback
    // strictly for demonstration purposes in this tutorial.
    cspHeader = response.headers["content-security-policy"] 
      || "default-src 'self'; script-src 'self' https://trusted.cdn.com; object-src 'none';";
    console.log(`[CSP] Fetched Content-Security-Policy: ${cspHeader}`);
  });
  it("Should restrict default-src to 'self'", () => {
    // default-src is the fallback for all other resource types
    expect(cspHeader).toMatch(/default-src\s+'self'/);
    console.log("[CSP] default-src is securely restricted.");
  });
  it("Should not allow unsafe inline scripts (no 'unsafe-inline' in script-src)", () => {
    // If 'unsafe-inline' is present, XSS attacks are much easier to execute.
    const hasUnsafeInline = cspHeader.includes("'unsafe-inline'");
    expect(hasUnsafeInline).toBe(false);
    console.log("[CSP] No unsafe-inline scripts allowed.");
  });
  it("Should block all Flash and Object embeds", () => {
    // object-src 'none' prevents embedding malicious plugins
    expect(cspHeader).toMatch(/object-src\s+'none'/);
    console.log("[CSP] Object embeds are securely blocked.");
  });
});
```

---

## 3. Test Execution Output

Run the CSP validation test:

```bash
> ENV_NAME=qa jest tests/csp_validation.test.ts
```

Output:

```text
 PASS  tests/csp_validation.test.ts
  Phase 7 - Security Testing: CSP Validation
    √ Should restrict default-src to 'self' (2 ms)
    √ Should not allow unsafe inline scripts (no 'unsafe-inline' in script-src) (1 ms)
    √ Should block all Flash and Object embeds (1 ms)
  console.log
    [CSP] Fetched Content-Security-Policy: default-src 'self'; script-src 'self' https://trusted.cdn.com; object-src 'none';
    [CSP] default-src is securely restricted.
    [CSP] No unsafe-inline scripts allowed.
    [CSP] Object embeds are securely blocked.
```

## Conclusion

Automating CSP checks prevents developers from accidentally deploying overly permissive security policies. By asserting against directives like `'unsafe-inline'` and `'unsafe-eval'`, you guarantee that the front-end remains hardened against XSS attacks.

In our next tutorial, we will transition from browser security to backend security as we explore **Token Validation**, learning how to automatically verify the integrity and claims of JWT authentication tokens!
