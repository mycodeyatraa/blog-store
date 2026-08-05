---
id: "post-797"
title: "NUnit/xUnit Test Lifecycle & Shared Context in Playwright C#"
slug: "nunit-xunit-test-lifecycle-shared-context-playwright-csharp"
date: "28-Feb-2026"
author: "pankaj-kumar"
series: "playwright-csharp-test-design"
seriesOrder: 6
topic: "6. NUnit/xUnit Lifecycle & Test Fixtures"
tags: ["Playwright", "C#", "NUnit", "xUnit", "Lifecycle"]
category: "Automation Testing"
categories: ["Automation Testing", "Playwright C#", "Test Lifecycle"]
excerpt: "Master NUnit `[SetUp]` / `[OneTimeSetUp]` and xUnit `IClassFixture` lifecycle state sharing in Playwright C#."
readTime: "8 min read"
---

# NUnit/xUnit Test Lifecycle & Shared Context in Playwright C#

Managing shared browser contexts and class-level initialization hooks properly balances test isolation with high-speed execution.

---

## 1. Lifecycle Hook Execution Flow

```
+---------------------------------------------------------------------------------+
| OneTimeSetUp / IClassFixture Constructor (Launch Shared IBrowser)               |
+---------------------------------------------------------------------------------+
       |
       +---> SetUp / Test Method 1 (Create Isolated IBrowserContext & IPage)
       |
       +---> SetUp / Test Method 2 (Create Isolated IBrowserContext & IPage)
       |
+---------------------------------------------------------------------------------+
| OneTimeTearDown / IDisposable.Dispose() (Close Shared IBrowser)                 |
+---------------------------------------------------------------------------------+
```

---

## 2. NUnit Shared Context Test Class

```csharp
// SharedContextTest.cs
using Microsoft.Playwright;
using NUnit.Framework;
using System.Threading.Tasks;
 
[TestFixture]
public class SharedContextTest
{
    private static IPlaywright _playwright;
    private static IBrowser _browser;
    private IBrowserContext _context;
    protected IPage Page { get; private set; }
 
    [OneTimeSetUp]
    public static async Task GlobalClassSetup()
    {
        _playwright = await Playwright.CreateAsync();
        _browser = await _playwright.Chromium.LaunchAsync(new BrowserTypeLaunchOptions { Headless = true });
    }
 
    [SetUp]
    public async Task PerTestSetup()
    {
        _context = await _browser.NewContextAsync();
        Page = await _context.NewPageAsync();
    }
 
    [Test]
    public async Task TestInIsolatedContext()
    {
        await Page.GotoAsync("https://mycodeyatra.com");
        Assert.That(await Page.TitleAsync(), Is.EqualTo("MyCodeYatra"));
    }
 
    [TearDown]
    public async Task PerTestTearDown()
    {
        await _context.CloseAsync();
    }
 
    [OneTimeTearDown]
    public static async Task GlobalClassTearDown()
    {
        await _browser.CloseAsync();
        _playwright.Dispose();
    }
}
```

---

## 3. Enterprise Best Practices & Comparison Summary

- **Class-Level Browser Launch**: Launch `IBrowser` once per class inside `OneTimeSetUp` for faster suite execution.
- **Per-Test Context Isolation**: Always create fresh `IBrowserContext` instances per test to prevent cookie/session bleeding.
