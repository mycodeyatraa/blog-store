---
id: "post-775"
title: "Browser Lifecycle & Resource Management in C#"
slug: "browser-lifecycle-and-resource-management-csharp"
date: "06-Feb-2026"
author: "pankaj-kumar"
series: "playwright-csharp-foundations"
seriesOrder: 5
topic: "5. Browser Lifecycle"
tags: ["Playwright", "C#", "Lifecycle", "Resource Management", "Dispose"]
category: "Automation Testing"
categories: ["Automation Testing", "Playwright C#", "Lifecycle"]
excerpt: "Manage Playwright C# browser lifecycles cleanly using `await using` resource disposal and NUnit setup/teardown hooks."
readTime: "8 min read"
---

# Browser Lifecycle & Resource Management in C#

Proper resource management prevents lingering node driver processes and memory leaks during large test suite executions.

---

## 1. Browser Lifecycle Diagram

```
+-----------------------------------+
| Playwright.CreateAsync()          |
+-----------------------------------+
                  |
                  v Launch Browser
+-----------------------------------+
| Chromium.LaunchAsync()            |
+-----------------------------------+
                  |
                  v Create Page & Execute Test
+-----------------------------------+
| Browser.NewContextAsync() -> Page |
+-----------------------------------+
                  |
                  v Clean Disposal
+-----------------------------------+
| await browser.CloseAsync()        |
| playwright.Dispose()              |
+-----------------------------------+
```

---

## 2. Resource Management Test Implementation

```csharp
// ResourceLifecycleTest.cs
using Microsoft.Playwright;
using NUnit.Framework;
using System;
using System.Threading.Tasks;
 
[TestFixture]
public class ResourceLifecycleTest
{
    private IPlaywright _playwright;
    private IBrowser _browser;
 
    [OneTimeSetUp]
    public async Task GlobalSetup()
    {
        _playwright = await Playwright.CreateAsync();
        _browser = await _playwright.Chromium.LaunchAsync(new BrowserTypeLaunchOptions { Headless = true });
    }
 
    [Test]
    public async Task TestWithLocalContextDisposal()
    {
        await using var context = await _browser.NewContextAsync();
        var page = await context.NewPageAsync();
 
        await page.GotoAsync("https://mycodeyatra.com");
        Assert.That(await page.TitleAsync(), Is.EqualTo("MyCodeYatra"));
    }
 
    [OneTimeTearDown]
    public async Task GlobalTeardown()
    {
        if (_browser != null) await _browser.CloseAsync();
        _playwright?.Dispose();
    }
}
```

---

## 3. Enterprise Best Practices & Comparison Summary

- **`IAsyncDisposable` Usage**: Use `await using` for `IBrowserContext` objects to ensure automatic cleanup upon method exit.
- **`OneTimeSetUp` Optimization**: Launch the browser once per test class in `OneTimeSetUp` and spawn fresh contexts per test in `SetUp`.
