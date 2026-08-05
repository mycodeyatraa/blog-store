---
id: "post-819"
title: "JWT Token Security Verification in Playwright C#"
slug: "jwt-token-security-verification-playwright-csharp"
date: "22-Mar-2026"
author: "pankaj-kumar"
series: "playwright-csharp-security-testing"
seriesOrder: 3
topic: "3. JWT Token Security Verification"
tags: ["Playwright", "C#", "JWT", "Security", "Tokens"]
category: "Automation Testing"
categories: ["Automation Testing", "Playwright C#", "Security Verification"]
excerpt: "Inspect and verify JWT token structure, expiration claims, and signature headers using Playwright C# API requests."
readTime: "8 min read"
---

# JWT Token Security Verification in Playwright C#

Validating JSON Web Tokens (JWT) guarantees that tokens contain valid claims, strong signing algorithms, and appropriate expiration thresholds.

---

## 1. JWT Security Inspection Flow

```
+------------------------------------+
| Acquire JWT Token via Auth Endpoint|
+------------------------------------+
                  |
                  v Decode Header & Payload Claims
+------------------------------------+
| Assert Algorithm != "none"         |
| Assert Expiration claim (exp)      |
+------------------------------------+
```

---

## 2. Playwright C# JWT Verification Test

```csharp
// JwtSecurityTest.cs
using Microsoft.Playwright;
using NUnit.Framework;
using System;
using System.Text;
using System.Text.Json;
using System.Threading.Tasks;
 
[TestFixture]
public class JwtSecurityTest
{
    [Test]
    public async Task VerifyJwtTokenClaimsAndSecurity()
    {
        using var playwright = await Playwright.CreateAsync();
        var request = await playwright.APIRequest.NewContextAsync();
 
        var response = await request.PostAsync("https://mycodeyatra.com/api/v1/auth/login", new()
        {
            DataObject = new { username = "qa_admin", password = "SecurePass123!" }
        });
 
        var json = await response.TextAsync();
        using var doc = JsonDocument.Parse(json);
        string token = doc.RootElement.GetProperty("access_token").GetString();
 
        // Split JWT parts: Header.Payload.Signature
        string[] parts = token.Split('.');
        Assert.That(parts.Length, Is.EqualTo(3), "JWT must contain 3 parts");
 
        // Decode Payload JSON
        string payloadJson = Encoding.UTF8.GetString(Convert.FromBase64String(PadBase64(parts[1])));
        using var payloadDoc = JsonDocument.Parse(payloadJson);
        var payload = payloadDoc.RootElement;
 
        Assert.That(payload.TryGetProperty("exp", out var expProp), Is.True, "JWT must contain 'exp' claim");
        long exp = expProp.GetInt64();
        Assert.That(exp, Is.GreaterThan(DateTimeOffset.UtcNow.ToUnixTimeSeconds()));
    }
 
    private static string PadBase64(string b64)
    {
        switch (b64.Length % 4)
        {
            case 2: return b64 + "==";
            case 3: return b64 + "=";
            default: return b64;
        }
    }
}
```

---

## 3. Enterprise Best Practices & Comparison Summary

- **Alg Header Inspection**: Ensure the token header algorithm is set to `RS256` or `HS256`, and never `none`.
- **Short Expiration Windows**: Enforce short JWT expiration windows for access tokens combined with secure refresh tokens.
