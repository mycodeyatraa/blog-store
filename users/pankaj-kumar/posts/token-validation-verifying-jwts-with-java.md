---
title: Token Validation: Verifying JWTs with Java
date: 15-Sep-2024
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [selenium, java, security-testing, jwt, jjwt, authentication]
category: UI Automation
categories: [UI Automation, Selenium, Java]
excerpt: >-
  Shift-left your security testing by cryptographically validating JSON Web Tokens (JWTs) using the JJWT library. Verify token expiration, signatures, and prevent PII exposure.
readTime: 5 min read
---

# Token Validation: Verifying JWTs with Java

In Phase 6, we learned how to extract a JSON Web Token (JWT) from the browser's Local Storage so we could pass it to RestAssured for API testing. But extracting a token is only half the battle. From a **Security Testing** perspective, how do we know the token is actually secure?

What if the token has no expiration date? What if it's using a weak hashing algorithm? What if the claims are unencrypted and expose sensitive PII (Personally Identifiable Information)?

In this article, we will learn how to shift-left our security testing by cryptographically validating JWTs directly within our Java test framework.

---

## 1. Anatomy of a JWT

A JWT is not encrypted; it is merely encoded in Base64. It consists of three parts separated by dots (`.`):
1. **Header:** Contains the algorithm used (e.g., `HS256` or `RS256`).
2. **Payload (Claims):** Contains the actual data (user ID, roles, expiration time).
3. **Signature:** A cryptographic hash of the Header + Payload, signed with a secret key.

Because the payload is just Base64, anyone can decode it. Therefore, a secure token must **never** contain passwords or SSNs, and it must have a strict expiration time (`exp` claim).

---

## 2. Setting Up the Java JWT Library

To decode and validate these tokens mathematically, we will use the industry-standard `jjwt` library. Add these dependencies to your `pom.xml`:

```xml
<dependencies>
    <dependency>
        <groupId>io.jsonwebtoken</groupId>
        <artifactId>jjwt-api</artifactId>
        <version>0.11.5</version>
    </dependency>
    <dependency>
        <groupId>io.jsonwebtoken</groupId>
        <artifactId>jjwt-impl</artifactId>
        <version>0.11.5</version>
        <scope>runtime</scope>
    </dependency>
    <dependency>
        <groupId>io.jsonwebtoken</groupId>
        <artifactId>jjwt-jackson</artifactId>
        <version>0.11.5</version>
        <scope>runtime</scope>
    </dependency>
</dependencies>
```

---

## 3. Automating JWT Security Checks

Let's write a test that extracts the JWT from the UI, parses it, and runs a battery of security assertions against its payload.

```java
package com.mycodeyatra.security;
 
import io.jsonwebtoken.Claims;
import io.jsonwebtoken.Jwts;
import org.testng.Assert;
import org.testng.annotations.Test;
 
import java.util.Date;
 
public class JwtSecurityTest {
 
    @Test
    public void verifyTokenSecurityCompliance() {
 
        // 1. Assume we extracted this token from the UI (as shown in Phase 6)
        String extractedJwt = "eyJhbGciOiJIUzI1NiIsInR5cCI... (Truncated)";
 
        // 2. Parse the token (We use parseClaimsJwt for unsigned parsing just to inspect the payload)
        // Note: To verify the *signature*, you would use parseClaimsJws(token) and provide the public key.
        // For basic structure testing from the client side, we strip the signature temporarily.
        String tokenWithoutSignature = extractedJwt.substring(0, extractedJwt.lastIndexOf('.') + 1);
 
        Claims claims = Jwts.parserBuilder()
                .build()
                .parseClaimsJwt(tokenWithoutSignature)
                .getBody();
 
        System.out.println("Token Subject: " + claims.getSubject());
        System.out.println("Token Expiration: " + claims.getExpiration());
 
        // 3. Security Assertion: Token MUST have an expiration date
        Assert.assertNotNull(claims.getExpiration(), "SECURITY FLAW: Token does not expire!");
 
        // 4. Security Assertion: Token lifespan must be short (e.g., < 60 minutes)
        Date issuedAt = claims.getIssuedAt();
        Date expiresAt = claims.getExpiration();
 
        long lifespanInMillis = expiresAt.getTime() - issuedAt.getTime();
        long lifespanInMinutes = lifespanInMillis / (60 * 1000);
 
        Assert.assertTrue(lifespanInMinutes <= 60, 
            "SECURITY FLAW: Token lifespan is too long (" + lifespanInMinutes + " mins). Max allowed is 60.");
 
        // 5. Security Assertion: No PII in Payload
        Assert.assertNull(claims.get("password"), "CRITICAL FLAW: Password exposed in JWT!");
        Assert.assertNull(claims.get("ssn"), "CRITICAL FLAW: SSN exposed in JWT!");
 
        // 6. Security Assertion: Correct Issuer
        Assert.assertEquals(claims.getIssuer(), "https://auth.mycodeyatra.com", 
            "SECURITY FLAW: Invalid Token Issuer!");
    }
}
```

---

## 4. Validating the Cryptographic Signature

The test above simply reads the Base64 data to verify compliance rules (lifespan, missing PII, issuer). However, the ultimate security test is verifying the **Signature**. 

If you have access to the backend's Public Key (for RS256) or Secret Key (for HS256), you can instruct the JJWT library to cryptographically verify that the token was not tampered with by a man-in-the-middle:

```java
import io.jsonwebtoken.security.Keys;
import java.security.Key;
 
public void verifyTokenSignature(String token) {
    // Note: The key should be loaded securely from an environment variable!
    String secret = "MySuperSecretKeyThatIsAtLeast32BytesLong123!";
    Key key = Keys.hmacShaKeyFor(secret.getBytes());
 
    try {
        // This will throw a SignatureException if the token was tampered with!
        Jwts.parserBuilder()
            .setSigningKey(key)
            .build()
            .parseClaimsJws(token);
 
        System.out.println("Signature is mathematically valid.");
    } catch (Exception e) {
        Assert.fail("SECURITY FLAW: Token signature is invalid or tampered with!");
    }
}
```

---

## System Architecture

Here is the flow of automated JWT validation within an enterprise pipeline:

![diagram_1](https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/posts/token-validation-verifying-jwts-with-java/images/diagram_1.png)

## Conclusion

Automated token validation guarantees that your backend engineers are following strict OAuth/JWT security guidelines. By asserting short lifespans, correct issuers, and ensuring no sensitive data is leaked into the Base64 payload, your Selenium framework becomes an active participant in your organization's security posture.

In our next article, we will take our security testing to the absolute maximum by integrating **OWASP ZAP (Zed Attack Proxy)** directly into our Java automation framework to run automated vulnerability scans while our UI tests execute!
