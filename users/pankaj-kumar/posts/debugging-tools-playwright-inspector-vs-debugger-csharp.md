---
id: "post-779"
title: "Debugging Tools: Playwright Inspector & VS Debugger in C#"
slug: "debugging-tools-playwright-inspector-vs-debugger-csharp"
date: "10-Feb-2026"
author: "pankaj-kumar"
series: "playwright-csharp-foundations"
seriesOrder: 9
topic: "9. Debugging Tools"
tags: ["Playwright", "C#", "Debugging", "Inspector", "Trace Viewer"]
category: "Automation Testing"
categories: ["Automation Testing", "Playwright C#", "Debugging"]
excerpt: "Debug Playwright C# tests using Playwright Inspector, Trace Viewer, and Visual Studio breakpoint debugging."
readTime: "8 min read"
---

# Debugging Tools: Playwright Inspector & VS Debugger in C#

Debugging failed automation scripts efficiently requires utilizing Playwright's built-in Inspector and Trace Viewer alongside Visual Studio.

---

## 1. Debugging Workflow Architecture

```
+------------------------------------+
| Execute `PWDEBUG=1 dotnet test`    |
+------------------------------------+
                  |
                  v Opens GUI Inspector
+------------------------------------+
| Playwright Inspector Window        |
| (Step Through, Pick Locator, DOM)  |
+------------------------------------+
                  |
                  v Inspect Failures Post-Run
+------------------------------------+
| Playwright Trace Viewer            |
| (pwsh playwright.ps1 show-trace)   |
+------------------------------------+
```

---

## 2. Playwright C# Trace Recording Code

```csharp
// DebuggingTest.cs
using Microsoft.Playwright;
using NUnit.Framework;
using System.Threading.Tasks;
 
[TestFixture]
public class DebuggingTest
{
    [Test]
    public async Task RecordTraceForDebugging()
    {
        using var playwright = await Playwright.CreateAsync();
        await using var browser = await playwright.Chromium.LaunchAsync(new BrowserTypeLaunchOptions { Headless = true });
        
        var context = await browser.NewContextAsync();
        
        // Start Tracing
        await context.Tracing.StartAsync(new TracingStartOptions
        {
            Screenshots = true,
            Snapshots = true,
            Sources = true
        });
 
        var page = await context.NewPageAsync();
        await page.GotoAsync("https://mycodeyatra.com");
 
        // Stop Tracing and save to zip
        await context.Tracing.StopAsync(new TracingStopOptions
        {
            Path = "target/traces/debug_trace.zip"
        });
    }
}
```

---

## 3. Enterprise Best Practices & Comparison Summary

- **PWDEBUG Environment Variable**: Set `PWDEBUG=1` in Terminal to launch Playwright Inspector automatically during execution.
- **Trace Viewer CLI**: Open saved traces using `pwsh bin/Debug/net8.0/playwright.ps1 show-trace target/traces/debug_trace.zip`.
