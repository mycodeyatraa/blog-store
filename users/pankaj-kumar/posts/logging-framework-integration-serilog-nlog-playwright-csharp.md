---
id: "post-800"
title: "Logging Framework Integration with Serilog & NLog in Playwright C#"
slug: "logging-framework-integration-serilog-nlog-playwright-csharp"
date: "03-Mar-2026"
author: "pankaj-kumar"
series: "playwright-csharp-test-design"
seriesOrder: 9
topic: "9. Logging Framework Integration with Serilog & NLog"
tags: ["Playwright", "C#", "Serilog", "NLog", "Logging"]
category: "Automation Testing"
categories: ["Automation Testing", "Playwright C#", "Logging"]
excerpt: "Integrate Serilog structured logging into Playwright C# test frameworks for console, file, and JSON log outputs."
readTime: "8 min read"
---

# Logging Framework Integration with Serilog & NLog in Playwright C#

Structured logging using Serilog captures execution timelines and parameter values, simplifying post-run test debugging.

---

## 1. Serilog Logging Pipeline

```
+------------------------------------+
| Test Execution Steps               |
+------------------------------------+
                  |
                  v Log.Information(...)
+---------------------------------------------------------------------------------+
| Serilog Sinks (Console Sink + File Sink: target/logs/automation.log)            |
+---------------------------------------------------------------------------------+
```

---

## 2. Serilog Logger Configuration

```csharp
// LoggerSetup.cs
using Serilog;
using System.IO;
 
namespace MyCodeYatra.Logging;
 
public static class LoggerSetup
{
    public static void InitializeLogger()
    {
        Log.Logger = new LoggerConfiguration()
            .MinimumLevel.Debug()
            .WriteTo.Console()
            .WriteTo.File("target/logs/test_execution.log", rollingInterval: RollingInterval.Day)
            .CreateLogger();
    }
}
```

---

## 3. Playwright Test with Serilog Logging

```csharp
// SerilogTest.cs
using Microsoft.Playwright.NUnit;
using MyCodeYatra.Logging;
using NUnit.Framework;
using Serilog;
using System.Threading.Tasks;
 
namespace MyCodeYatra.PlaywrightTests;
 
[TestFixture]
public class SerilogTest : PageTest
{
    [OneTimeSetUp]
    public void SetupLogger()
    {
        LoggerSetup.InitializeLogger();
        Log.Information("Starting Playwright Test Execution Suite");
    }
 
    [Test]
    public async Task TestLoginWithStructuredLogging()
    {
        Log.Information("Navigating to login page");
        await Page.GotoAsync("https://mycodeyatra.com/login");
 
        Log.Information("Filling credentials for user {Username}", "qa_admin");
        await Page.GetByLabel("Username").FillAsync("qa_admin");
        await Page.GetByLabel("Password").FillAsync("Secret123!");
        await Page.GetByRole(AriaRole.Button, new() { Name = "Sign In" }).ClickAsync();
 
        Log.Information("Asserting dashboard visibility");
        await Expect(Page.Locator(".welcome-banner")).ToBeVisibleAsync();
    }
 
    [OneTimeTearDown]
    public void CloseLogger()
    {
        Log.CloseAndFlush();
    }
}
```

---

## 4. Enterprise Best Practices & Comparison Summary

- **Structured Log Messages**: Use message templates (`Log.Information("User {Username} logged in", user)`) to enable log searching engines to parse values easily.
- **Log Flushing**: Call `Log.CloseAndFlush()` in `OneTimeTearDown` to ensure all pending logs write to disk before process exit.
