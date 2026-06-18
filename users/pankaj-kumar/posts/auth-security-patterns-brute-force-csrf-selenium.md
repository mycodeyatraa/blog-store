---
title: Hardening the Gates: Automating Authentication Security Patterns
date: 25-Dec-2024
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [typescript, selenium, security, authentication, brute-force, csrf, lockout]
category: Selenium TypeScript
categories: [Selenium TypeScript, Security Testing Automation]
excerpt: >-
  In the grand finale of our series, learn how to automate tests against modern security mechanisms like Brute Force Account Lockouts and CSRF tokens.
readTime: 4 min read
---

# Hardening the Gates: Automating Authentication Security Patterns

Welcome to the grand finale of our Selenium TypeScript curriculum! Over the last 47 posts, we have journeyed from basic element locators to complex Page Object Models, API/UI Hybrid frameworks, and Dynamic Security Testing.

In this final tutorial, we bring everything together to test the most critical barrier of any application: **Authentication Security**.

Specifically, we will automate tests for two fundamental security patterns:
1. **Brute Force Protection (Account Lockout)**
2. **Cross-Site Request Forgery (CSRF) Prevention**

---

## 1. Testing for Brute Force Protection

A Brute Force attack occurs when a hacker uses an automated script to guess a user's password thousands of times per second. To prevent this, modern applications implement **Account Lockout Policies** (e.g., locking the account for 15 minutes after 5 failed attempts) and respond with an HTTP `429 Too Many Requests` status.

We can easily automate this simulation using Axios!

Create a new file `tests/auth_security.test.ts`:

```typescript
import axios from "axios";
describe("Phase 7 - Security Testing: Auth Security Patterns", () => {
  const TARGET_URL = "https://practice.mycodeyatra.com/";
  const LOGIN_API = `${TARGET_URL}api/login`; 
  it("Should enforce Account Lockout after 5 failed login attempts (Brute Force Protection)", async () => {
    const testUser = "bruteforce_test@example.com";
    const invalidPassword = "WrongPassword123!";
    let finalStatus = 200;
    console.log(`[Security] Initiating Brute Force simulation against ${testUser}...`);
    for (let i = 1; i <= 6; i++) {
      try {
        await axios.post(LOGIN_API, { email: testUser, password: invalidPassword });
      } catch (error: any) {
        finalStatus = error.response?.status || 500;
        if (i <= 5) {
          // The first 5 attempts should return 401 Unauthorized
          expect(finalStatus).toBe(401);
        } else {
          // The 6th attempt should trigger the security mechanism
          // 429 = Too Many Requests (Rate Limiting / Account Lockout)
          expect(finalStatus).toBe(429);
          console.log(`[Security] Account successfully locked on attempt ${i}. Status: 429.`);
        }
      }
    }
  });
```

---

## 2. Validating CSRF Token Presence

Cross-Site Request Forgery (CSRF) tricks a logged-in user's browser into executing unwanted actions (like transferring funds) on a different site. 

The industry standard defense is the **Anti-CSRF Token**—a unique, unguessable string embedded in hidden HTML `<input>` fields for every state-changing form (POST/PUT/DELETE).

We can automate a check to ensure developers haven't forgotten to include this token in their forms:

```typescript
  it("Should return a valid CSRF token in state-changing forms", async () => {
    console.log("[Security] Fetching login page HTML to extract CSRF token...");
    const response = await axios.get(TARGET_URL);
    const htmlBody = response.data;
    // We assert that a hidden CSRF field exists in the raw HTML payload
    // Example: <input type="hidden" name="_csrf" value="a8b7c6d5...">
    const hasCsrfField = htmlBody.includes('name="_csrf"') || htmlBody.includes('name="csrf-token"');
    expect(hasCsrfField).toBe(true);
    console.log(`[Security] CSRF Protection Check Passed: Found hidden CSRF token.`);
  });
});
```

---

## 3. Test Execution Output

Run the final security script:

```bash
> ENV_NAME=qa jest tests/auth_security.test.ts
```

Output:

```text
 PASS  tests/auth_security.test.ts
  Phase 7 - Security Testing: Auth Security Patterns
    √ Should enforce Account Lockout after 5 failed login attempts (502 ms)
    √ Should return a valid CSRF token in state-changing forms (42 ms)
  console.log
    [Security] Initiating Brute Force simulation against bruteforce_test@example.com...
    [Security] Account successfully locked on attempt 6. Status: 429.
    [Security] Fetching login page HTML to extract CSRF token...
    [Security] CSRF Protection Check Passed: Found hidden CSRF token.
```

## Curriculum Conclusion

Congratulations! Over the course of 48 extensive tutorials, you have mastered everything from the foundational architecture of Selenium WebDriver to the absolute bleeding edge of Hybrid API/UI interactions and Automated Security Testing.

You are no longer just an automation engineer; you are an **SDET** (Software Development Engineer in Test) capable of building robust, scalable, and secure enterprise test frameworks. 

Thank you for following the `TEST_SEL_TYPESCRIPT` roadmap at MyCodeYatra! Keep automating, keep securing, and happy coding!
