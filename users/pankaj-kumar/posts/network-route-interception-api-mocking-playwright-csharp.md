---
id: "post-811"
title: "Network Route Interception & API Mocking in Playwright C#"
slug: "network-route-interception-api-mocking-playwright-csharp"
date: "14-Mar-2026"
author: "pankaj-kumar"
series: "playwright-csharp-api-testing"
seriesOrder: 8
topic: "8. Network Route Interception & API Mocking"
tags: ["Playwright", "C#", "Route Interception", "API Mocking", "Network"]
category: "Automation Testing"
categories: ["Automation Testing", "Playwright C#", "Mocking"]
excerpt: "Intercept browser HTTP traffic and inject mock JSON responses using `Page.RouteAsync` in Playwright C#."
readTime: "8 min read"
---

# Network Route Interception & API Mocking in Playwright C#

Mocking backend API responses allows frontend UI components to be tested independently of backend service availability.

---

## 1. Mocking Architecture

```
+------------------------------------+       +------------------------------------+
| Playwright Page Context            | ----> | Intercept Route                    |
| Network Request Triggered          |       | Page.RouteAsync("**/api/user")    |
+------------------------------------+       +------------------------------------+
                                                        |
                                                        v Fulfill Fake Response
+---------------------------------------------------------------------------------+
| Route.FulfillAsync(Status 200, Mock JSON Payload)                               |
+---------------------------------------------------------------------------------+
```

---

## 2. Playwright C# Mocking Test

```csharp
// RouteMockTest.cs
using Microsoft.Playwright;
using Microsoft.Playwright.NUnit;
using NUnit.Framework;
using System.Threading.Tasks;
 
namespace MyCodeYatra.PlaywrightTests;
 
[TestFixture]
public class RouteMockTest : PageTest
{
    [Test]
    public async Task MockApiResponseForFrontendValidation()
    {
        // Intercept target API endpoint and fulfill fake payload
        await Page.RouteAsync("**/api/v1/dashboard/stats", async route =>
        {
            await route.FulfillAsync(new RouteFulfillOptions
            {
                Status = 200,
                ContentType = "application/json",
                Body = "{"totalUsers": 9999, "activeServers": 42}"
            });
        });
 
        await Page.GotoAsync("https://mycodeyatra.com/dashboard");
        await Expect(Page.Locator("#user-count")).ToHaveTextAsync("9999");
    }
}
```

---

## 3. Enterprise Best Practices & Comparison Summary

- **Isolated Mocks**: Mock flaky third-party APIs (payment gateways, weather services) to ensure reliable UI test runs.
- **Fallback Unhandled**: Call `await route.ContinueAsync()` for unhandled routes to allow normal network traffic flow.
