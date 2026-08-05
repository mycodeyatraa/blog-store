---
id: "post-804"
title: "APIRequestContext Fundamentals in Playwright C#"
slug: "apirequestcontext-fundamentals-playwright-csharp"
date: "07-Mar-2026"
author: "pankaj-kumar"
series: "playwright-csharp-api-testing"
seriesOrder: 1
topic: "1. APIRequestContext Fundamentals in C#"
tags: ["Playwright", "C#", "APIRequestContext", "REST API", "HTTP"]
category: "Automation Testing"
categories: ["Automation Testing", "Playwright C#", "API Automation"]
excerpt: "Execute high-speed HTTP REST API requests using Playwright C# IAPIRequestContext without third-party HTTP clients."
readTime: "8 min read"
---

# APIRequestContext Fundamentals in Playwright C#

Playwright C# includes native `IAPIRequestContext` capabilities, enabling direct HTTP requests to REST APIs without extra dependencies like RestSharp or HttpClient.

---

## 1. API Architecture Overview

`IAPIRequestContext` manages HTTP headers, base URLs, and authentication context directly within Playwright instances.

```
+------------------------------------+       +------------------------------------+
| Playwright C# Test                 | ----> | Playwright IAPIRequestContext      |
| (APIRequestContext)                |       | (Manages Headers, Auth, Cookies)   |
+------------------------------------+       +------------------------------------+
                                                        |
                                                        v Direct HTTP Request
+---------------------------------------------------------------------------------+
| Backend Microservices / REST Endpoints                                          |
+---------------------------------------------------------------------------------+
```

---

## 2. Playwright C# API Context Setup Code

```csharp
// ApiFundamentalsTest.cs
using Microsoft.Playwright;
using NUnit.Framework;
using System.Collections.Generic;
using System.Threading.Tasks;
 
[TestFixture]
public class ApiFundamentalsTest
{
    private IPlaywright _playwright;
    private IAPIRequestContext _request;
 
    [SetUp]
    public async Task Setup()
    {
        _playwright = await Playwright.CreateAsync();
        _request = await _playwright.APIRequest.NewContextAsync(new APIRequestNewContextOptions
        {
            BaseURL = "https://mycodeyatra.com",
            ExtraHTTPHeaders = new Dictionary<string, string>
            {
                { "Accept", "application/json" },
                { "Authorization", "Bearer sample_token_123" }
            }
        });
    }
 
    [Test]
    public async Task VerifyApiHealthEndpoint()
    {
        var response = await _request.GetAsync("/api/v1/health");
        Assert.That(response.Status, Is.EqualTo(200));
        Assert.That(await response.TextAsync(), Contains.Substring("OK"));
    }
 
    [TearDown]
    public async Task TearDown()
    {
        await _request.DisposeAsync();
        _playwright.Dispose();
    }
}
```

---

## 3. Enterprise Best Practices & Comparison Summary

- **Built-in Session State**: Shares cookie and storage state between UI and API request contexts seamlessly.
- **Resource Cleanup**: Always call `await _request.DisposeAsync()` in teardown blocks to free HTTP connections.
