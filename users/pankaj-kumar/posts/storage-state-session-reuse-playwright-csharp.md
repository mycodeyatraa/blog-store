---
id: "post-814"
title: "Storage State Session Reuse in Playwright C#"
slug: "storage-state-session-reuse-playwright-csharp"
date: "17-Mar-2026"
author: "pankaj-kumar"
series: "playwright-csharp-api-testing"
seriesOrder: 11
topic: "11. Storage State Session Reuse"
tags: ["Playwright", "C#", "Storage State", "Session Reuse", "Auth"]
category: "Automation Testing"
categories: ["Automation Testing", "Playwright C#", "Storage State"]
excerpt: "Save and load authenticated browser storage state JSON files in Playwright C# to bypass login UI steps."
readTime: "8 min read"
---

# Storage State Session Reuse in Playwright C#

Saving authenticated browser cookies and localStorage into a `storageState.json` file allows test suites to bypass login UIs across hundreds of tests.

---

## 1. Storage State Workflow

```
+------------------------------------+
| 1. Execute One-Time Login          |
+------------------------------------+
                  |
                  v Save Storage State
+------------------------------------+
| state.json (Cookies & LocalStorage)|
+------------------------------------+
                  |
                  v Load in New Contexts
+------------------------------------+
| NewContextAsync(StorageStatePath)  |
+------------------------------------+
```

---

## 2. Playwright C# Storage State Test

```csharp
// StorageStateTest.cs
using Microsoft.Playwright;
using Microsoft.Playwright.NUnit;
using NUnit.Framework;
using System.IO;
using System.Threading.Tasks;
 
namespace MyCodeYatra.PlaywrightTests;
 
[TestFixture]
public class StorageStateTest
{
    private const string StateFilePath = "target/auth/state.json";
 
    [Test]
    public async Task Step1_SaveStorageState()
    {
        using var playwright = await Playwright.CreateAsync();
        await using var browser = await playwright.Chromium.LaunchAsync(new() { Headless = true });
 
        var context = await browser.NewContextAsync();
        var page = await context.NewPageAsync();
 
        await page.GotoAsync("https://mycodeyatra.com/login");
        await page.GetByLabel("Username").FillAsync("qa_admin");
        await page.GetByLabel("Password").FillAsync("AdminPass123!");
        await page.GetByRole(AriaRole.Button, new() { Name = "Sign In" }).ClickAsync();
 
        // Save authenticated storage state
        Directory.CreateDirectory("target/auth");
        await context.StorageStateAsync(new BrowserContextStorageStateOptions
        {
            Path = StateFilePath
        });
    }
 
    [Test]
    public async Task Step2_ReuseAuthenticatedStorageState()
    {
        using var playwright = await Playwright.CreateAsync();
        await using var browser = await playwright.Chromium.LaunchAsync(new() { Headless = true });
 
        // Load storage state directly
        var context = await browser.NewContextAsync(new BrowserNewContextOptions
        {
            StorageStatePath = StateFilePath
        });
 
        var page = await context.NewPageAsync();
        await page.GotoAsync("https://mycodeyatra.com/dashboard");
 
        // Verify pre-authenticated access
        Assert.That(await page.Locator(".user-profile").IsVisibleAsync(), Is.True);
    }
}
```

---

## 3. Enterprise Best Practices & Comparison Summary

- **Global Setup**: Perform storage state login generation once in NUnit `[OneTimeSetUp]` or global setup files.
- **Git Ignore Secrets**: Add `state.json` to `.gitignore` to prevent session token leaks.
