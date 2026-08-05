---
id: "post-794"
title: "Factory Pattern for Driver & Page Management in Playwright C#"
slug: "factory-pattern-driver-page-management-playwright-csharp"
date: "25-Feb-2026"
author: "pankaj-kumar"
series: "playwright-csharp-test-design"
seriesOrder: 3
topic: "3. Factory Pattern"
tags: ["Playwright", "C#", "Factory Pattern", "Browser Factory", "Architecture"]
category: "Automation Testing"
categories: ["Automation Testing", "Playwright C#", "Design Patterns"]
excerpt: "Implement Factory pattern classes in C# to manage browser engine instantiation and Page Object initialization."
readTime: "8 min read"
---

# Factory Pattern for Driver & Page Management in Playwright C#

The Factory design pattern centralizes browser launch configurations, headful/headless switches, and context instantiation logic in one place.

---

## 1. Factory Architecture

```
+----------------------------------+
| PlaywrightTestRunner             |
+----------------------------------+
                 |
                 v Invokes LoggerFactory
+----------------------------------+
| BrowserFactory.CreateBrowser()   |
| (Chromium / Firefox / WebKit)    |
+----------------------------------+
```

---

## 2. C# Browser Factory Implementation

```csharp
// BrowserFactory.cs
using Microsoft.Playwright;
using System;
using System.Threading.Tasks;
 
namespace MyCodeYatra.Factories;
 
public static class BrowserFactory
{
    public static async Task<IBrowser> CreateBrowserAsync(IPlaywright playwright, string browserType, bool headless = true)
    {
        var options = new BrowserTypeLaunchOptions { Headless = headless };
 
        return browserType.ToLower() switch
        {
            "firefox" => await playwright.Firefox.LaunchAsync(options),
            "webkit" => await playwright.WebKit.LaunchAsync(options),
            _ => await playwright.Chromium.LaunchAsync(options)
        };
    }
}
```

---

## 3. Playwright Factory Test

```csharp
// FactoryTest.cs
using Microsoft.Playwright;
using MyCodeYatra.Factories;
using NUnit.Framework;
using System.Threading.Tasks;
 
namespace MyCodeYatra.PlaywrightTests;
 
[TestFixture]
public class FactoryTest
{
    [Test]
    public async Task TestBrowserFactoryInstantiation()
    {
        using var playwright = await Playwright.CreateAsync();
        await using var browser = await BrowserFactory.CreateBrowserAsync(playwright, "chromium", headless: true);
 
        var page = await browser.NewPageAsync();
        await page.GotoAsync("https://mycodeyatra.com");
 
        Assert.That(await page.TitleAsync(), Is.EqualTo("MyCodeYatra"));
    }
}
```

---

## 4. Enterprise Best Practices & Comparison Summary

- **Centralized Launch Options**: Keep browser flags, proxy settings, and headless toggles inside `BrowserFactory`.
- **Environment Driven**: Read target browser parameters from `appsettings.json` or command-line environment variables.
