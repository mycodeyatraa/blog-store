---
id: "post-805"
title: "Validating GET API Requests in Playwright C#"
slug: "validating-get-api-requests-playwright-csharp"
date: "08-Mar-2026"
author: "pankaj-kumar"
series: "playwright-csharp-api-testing"
seriesOrder: 2
topic: "2. Validating GET API Requests"
tags: ["Playwright", "C#", "GET API", "JSON", "Schema Validation"]
category: "Automation Testing"
categories: ["Automation Testing", "Playwright C#", "API Automation"]
excerpt: "Execute and validate HTTP GET API responses, status codes, and JSON schemas in Playwright C#."
readTime: "8 min read"
---

# Validating GET API Requests in Playwright C#

HTTP GET endpoints retrieve resource representations. Validating response status codes, headers, and JSON body attributes ensures API contract compliance.

---

## 1. GET API Validation Flow

```
+------------------------------------+
| _request.GetAsync("/api/users/1") |
+------------------------------------+
                  |
                  v Returns IAPIResponse
+------------------------------------+
| 1. Assert Status: 200 OK           |
| 2. Deserialize JSON Body           |
| 3. Assert Attributes & Schema      |
+------------------------------------+
```

---

## 2. Playwright C# GET API Test

```csharp
// GetApiTest.cs
using Microsoft.Playwright;
using NUnit.Framework;
using System.Text.Json;
using System.Threading.Tasks;
 
public record UserProfileResponse(string Id, string Name, string Email);
 
[TestFixture]
public class GetApiTest
{
    [Test]
    public async Task ValidateGetUserProfileApi()
    {
        using var playwright = await Playwright.CreateAsync();
        var request = await playwright.APIRequest.NewContextAsync();
 
        var response = await request.GetAsync("https://mycodeyatra.com/api/v1/users/101");
        
        Assert.That(response.Status, Is.EqualTo(200));
        Assert.That(response.Headers["content-type"], Contains.Substring("application/json"));
 
        var jsonText = await response.TextAsync();
        var user = JsonSerializer.Deserialize<UserProfileResponse>(jsonText, new JsonSerializerOptions { PropertyNameCaseInsensitive = true });
 
        Assert.That(user.Id, Is.EqualTo("101"));
        Assert.That(user.Name, Is.EqualTo("Pankaj Kumar"));
    }
}
```

---

## 3. Enterprise Best Practices & Comparison Summary

- **Strongly Typed Deserialization**: Use System.Text.Json record types for fast, type-safe JSON assertions.
- **Header Inspection**: Verify Content-Type and Cache-Control headers explicitly to enforce REST standards.
