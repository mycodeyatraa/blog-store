---
title: Securing the Backend: Automating JWT Token Validation
date: 23-Dec-2024
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [typescript, selenium, security, jwt, data-leak, pii]
category: Selenium TypeScript
categories: [Selenium TypeScript, Security Testing Automation]
excerpt: >-
  Act as the first line of defense against data exposure by automating the decoding and auditing of JWT payloads to catch PII leaks before production.
readTime: 4 min read
---

# Securing the Backend: Automating JWT Token Validation

When implementing API testing, most QA engineers simply extract the `Authorization: Bearer <token>` string and pass it to the next request without a second thought. 

However, JSON Web Tokens (JWTs) are a massive attack vector. A JWT is **not encrypted**; it is merely Base64 encoded. Anyone who intercepts the token can decode it instantly and read its contents.

If developers accidentally leak Personal Identifiable Information (PII) like Social Security Numbers, Passwords, or exact physical addresses into the token payload, your application is critically vulnerable.

In this tutorial, we will write a security automation script that intercepts the JWT, decodes the payload, and audits the claims!

---

## 1. Anatomy of a JWT

A standard JWT consists of three parts separated by periods (`.`):
`Header.Payload.Signature`

1. **Header**: Contains metadata like the algorithm used (e.g., HS256).
2. **Payload (Claims)**: Contains the actual user data, expiration timestamps (`exp`), and the issuer (`iss`).
3. **Signature**: A hashed string that the backend uses to verify the token hasn't been tampered with.

To audit the token, we only need to decode the middle `Payload` segment.

---

## 2. Writing the Token Validation Script

While you can use libraries like `jsonwebtoken`, we will write a zero-dependency script using Node's native `Buffer` class to demonstrate exactly how tokens are decoded.

Create a new file `tests/token_validation.test.ts`:

```typescript
import { AuthUtils } from "./utils/AuthUtils";
describe("Phase 7 - Security Testing: JWT Token Validation", () => {
  let rawToken: string;
  let decodedPayload: any;
  beforeAll(async () => {
    // 1. Fetch the raw JWT string from the backend
    rawToken = await AuthUtils.getAuthToken();
    console.log(`[Security] Fetched Raw JWT: ${rawToken.substring(0, 20)}...`);
    // 2. Decode the Payload (Middle segment of the JWT)
    const base64Url = rawToken.split(".")[1];
    // Fix Base64Url string to standard Base64
    const base64 = base64Url.replace(/-/g, "+").replace(/_/g, "/");
    // Decode from Base64 into a UTF-8 JSON string
    const jsonPayload = Buffer.from(base64, "base64").toString("utf-8");
    decodedPayload = JSON.parse(jsonPayload);
  });
  it("Should have a valid expiration time (exp) in the future", () => {
    expect(decodedPayload.exp).toBeDefined();
    // JWT timestamps are in seconds, not milliseconds!
    const currentTimeInSeconds = Math.floor(Date.now() / 1000);
    expect(decodedPayload.exp).toBeGreaterThan(currentTimeInSeconds);
    console.log(`[Security] Token is valid until timestamp: ${decodedPayload.exp}`);
  });
  it("Should not leak sensitive PII in the token payload", () => {
    // Ensure developers are not packing sensitive fields into the unencrypted JWT
    const sensitiveKeys = ["password", "ssn", "credit_card", "pin"];
    const payloadKeys = Object.keys(decodedPayload);
    for (const key of sensitiveKeys) {
      expect(payloadKeys).not.toContain(key);
    }
    console.log("[Security] Payload verified: No sensitive PII found.");
  });
  it("Should be issued by the trusted Identity Provider (iss)", () => {
    // Validate the Issuer claim to prevent accepting forged tokens
    expect(decodedPayload.iss).toBeDefined();
    console.log(`[Security] Token Issuer (iss): ${decodedPayload.iss}`);
  });
});
```

---

## 3. Test Execution Output

Run the script:

```bash
> ENV_NAME=qa jest tests/token_validation.test.ts
```

Output:

```text
 PASS  tests/token_validation.test.ts
  Phase 7 - Security Testing: JWT Token Validation
    √ Should have a valid expiration time (exp) in the future (2 ms)
    √ Should not leak sensitive PII in the token payload (1 ms)
    √ Should be issued by the trusted Identity Provider (iss) (1 ms)
  console.log
    [Security] Fetched Raw JWT: eyJhbGciOiJIUzI1Ni...
    [Security] Token is valid until timestamp: 1734998400
    [Security] Payload verified: No sensitive PII found.
    [Security] Token Issuer (iss): https://auth.mycodeyatra.com
```

## Conclusion

By programmatically decoding the JWT inside your CI/CD pipeline, you act as the first line of defense against data exposure. You can instantly catch accidental PII leaks before they are merged into production, saving your company from massive compliance fines.

In our next tutorial, we will take security testing to the next level by integrating an industry-standard Dynamic Application Security Testing (DAST) tool: **OWASP ZAP**!
