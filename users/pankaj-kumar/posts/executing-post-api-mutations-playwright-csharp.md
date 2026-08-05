---
id: "post-806"
title: "Executing POST API Mutations in Playwright C#"
slug: "executing-post-api-mutations-playwright-csharp"
date: "09-Mar-2026"
author: "pankaj-kumar"
series: "playwright-csharp-api-testing"
seriesOrder: 3
topic: "3. Executing POST API Mutations"
tags: ["Playwright", "C#", "POST API", "Mutations", "JSON Payload"]
category: "Automation Testing"
categories: ["Automation Testing", "Playwright C#", "API Automation"]
excerpt: "Construct and send HTTP POST API requests with JSON payloads in Playwright C# for resource creation."
readTime: "8 min read"
---

# Executing POST API Mutations in Playwright C#

HTTP POST requests create backend resources. Supplying structured JSON payloads via Playwright's `DataObject` option validates successful resource creation.

---

## 1. POST Mutation Architecture

```
+---------------------------------------------------------------------------------+
| _request.PostAsync("/api/users", new() { DataObject = new { name = "Alex" } }) |
+---------------------------------------------------------------------------------+
                  |
                  v Returns 201 Created
+---------------------------------------------------------------------------------+
| Assert Response ID & Persistence                                               |
+---------------------------------------------------------------------------------+
```

---

## 2. Playwright C# POST API Test

```csharp
// PostApiTest.cs
using Microsoft.Playwright;
using NUnit.Framework;
using System.Text.Json;
using System.Threading.Tasks;
 
public record CreateUserRequest(string Name, string Email, string Role);
public record CreateUserResponse(string Id, string Status);
 
[TestFixture]
public class PostApiTest
{
    [Test]
    public async Task CreateNewUserViaPostApi()
    {
        using var playwright = await Playwright.CreateAsync();
        var request = await playwright.APIRequest.NewContextAsync();
 
        var newUser = new CreateUserRequest("Automation User", "user@mycodeyatra.com", "QA");
 
        var response = await request.PostAsync("https://mycodeyatra.com/api/v1/users", new APIRequestContextOptions
        {
            DataObject = newUser
        });
 
        Assert.That(response.Status, Is.EqualTo(201));
 
        var json = await response.TextAsync();
        var result = JsonSerializer.Deserialize<CreateUserResponse>(json, new JsonSerializerOptions { PropertyNameCaseInsensitive = true });
 
        Assert.That(result.Id, Is.Not.Null);
        Assert.That(result.Status, Is.EqualTo("CREATED"));
    }
}
```

---

## 3. Enterprise Best Practices & Comparison Summary

- **`DataObject` Serialization**: Pass C# anonymous objects or records directly into `DataObject` options for automatic JSON serialization.
- **Status Code 201**: Verify HTTP status 201 Created for resource generation operations.
