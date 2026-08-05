---
id: "post-813"
title: "Reusable API Framework Architecture in Playwright C#"
slug: "reusable-api-framework-architecture-playwright-csharp"
date: "16-Mar-2026"
author: "pankaj-kumar"
series: "playwright-csharp-api-testing"
seriesOrder: 10
topic: "10. Reusable API Framework Architecture"
tags: ["Playwright", "C#", "API Framework", "Service Clients", "Architecture"]
category: "Automation Testing"
categories: ["Automation Testing", "Playwright C#", "API Framework"]
excerpt: "Design clean API Service Client classes in C# to encapsulate REST endpoints and request serialization."
readTime: "8 min read"
---

# Reusable API Framework Architecture in Playwright C#

Structuring API automation code into dedicated Service Client classes decouples raw HTTP requests from test methods.

---

## 1. API Framework Architecture

```
+----------------------------------+
| UserApiTest.cs (Test Method)    |
+----------------------------------+
                 |
                 v Calls Service Client
+----------------------------------+
| UserApiClient.cs                 |
| (GetUser(), CreateUser())        |
+----------------------------------+
```

---

## 2. Reusable C# API Service Client

```csharp
// UserApiClient.cs
using Microsoft.Playwright;
using System.Threading.Tasks;
 
namespace MyCodeYatra.ApiClients;
 
public class UserApiClient
{
    private readonly IAPIRequestContext _request;
 
    public UserApiClient(IAPIRequestContext request)
    {
        _request = request;
    }
 
    public async Task<IAPIResponse> GetUserAsync(string userId)
    {
        return await _request.GetAsync($"/api/v1/users/{userId}");
    }
 
    public async Task<IAPIResponse> CreateUserAsync(object userPayload)
    {
        return await _request.PostAsync("/api/v1/users", new APIRequestContextOptions
        {
            DataObject = userPayload
        });
    }
}
```

---

## 3. Playwright API Client Test

```csharp
// ApiClientTest.cs
using Microsoft.Playwright;
using MyCodeYatra.ApiClients;
using NUnit.Framework;
using System.Threading.Tasks;
 
[TestFixture]
public class ApiClientTest
{
    [Test]
    public async Task TestUsingApiClient()
    {
        using var playwright = await Playwright.CreateAsync();
        var request = await playwright.APIRequest.NewContextAsync(new() { BaseURL = "https://mycodeyatra.com" });
 
        var client = new UserApiClient(request);
        var response = await client.GetUserAsync("101");
 
        Assert.That(response.Status, Is.EqualTo(200));
    }
}
```

---

## 4. Enterprise Best Practices & Comparison Summary

- **Service Client Encapsulation**: Keep endpoint URLs and HTTP methods encapsulated inside client classes.
- **Shared Request Context**: Pass a single `IAPIRequestContext` to multiple API client instances.
