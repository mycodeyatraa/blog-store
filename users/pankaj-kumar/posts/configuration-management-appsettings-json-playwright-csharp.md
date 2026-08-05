---
id: "post-799"
title: "Configuration Management with appsettings.json in Playwright C#"
slug: "configuration-management-appsettings-json-playwright-csharp"
date: "02-Mar-2026"
author: "pankaj-kumar"
series: "playwright-csharp-test-design"
seriesOrder: 8
topic: "8. Configuration Management with appsettings.json"
tags: ["Playwright", "C#", "appsettings.json", "Configuration", ".NET"]
category: "Automation Testing"
categories: ["Automation Testing", "Playwright C#", "Configuration"]
excerpt: "Manage environment settings, URLs, timeouts, and credentials using `appsettings.json` and `Microsoft.Extensions.Configuration` in C#."
readTime: "8 min read"
---

# Configuration Management with appsettings.json in Playwright C#

Managing environment settings cleanly using `appsettings.json` enables switching between Staging, QA, and Production environments seamlessly.

---

## 1. Configuration Layer Architecture

```
+------------------------------------+
| appsettings.json / appsettings.QA.json
+------------------------------------+
                  |
                  v Microsoft.Extensions.Configuration
+------------------------------------+
| TestSettings Model Object          |
+------------------------------------+
```

---

## 2. Configuration Settings Model & JSON File

```json
// appsettings.json
{
  "TestSettings": {
    "BaseUrl": "https://staging.mycodeyatra.com",
    "TimeoutMs": 15000,
    "Headless": true
  }
}
```

```csharp
// ConfigManager.cs
using Microsoft.Extensions.Configuration;
using System.IO;
 
namespace MyCodeYatra.Config;
 
public class TestSettings
{
    public string BaseUrl { get; set; }
    public float TimeoutMs { get; set; }
    public bool Headless { get; set; }
}
 
public static class ConfigManager
{
    public static TestSettings Settings { get; }
 
    static ConfigManager()
    {
        var config = new ConfigurationBuilder()
            .SetBasePath(Directory.GetCurrentDirectory())
            .AddJsonFile("appsettings.json", optional: false, reloadOnChange: true)
            .AddEnvironmentVariables()
            .Build();
 
        Settings = config.GetSection("TestSettings").Get<TestSettings>();
    }
}
```

---

## 3. Playwright Configuration Test

```csharp
// ConfigTest.cs
using Microsoft.Playwright.NUnit;
using MyCodeYatra.Config;
using NUnit.Framework;
using System.Threading.Tasks;
 
namespace MyCodeYatra.PlaywrightTests;
 
[TestFixture]
public class ConfigTest : PageTest
{
    [Test]
    public async Task TestNavigationUsingConfigSettings()
    {
        await Page.GotoAsync(ConfigManager.Settings.BaseUrl);
        await Expect(Page).ToHaveTitleAsync("MyCodeYatra");
    }
}
```

---

## 4. Enterprise Best Practices & Comparison Summary

- **Environment Override**: Use `.AddEnvironmentVariables()` to allow CI build pipelines to override `appsettings.json` parameters.
- **Never Commit Secrets**: Exclude sensitive passwords from `appsettings.json` and retrieve them from environment secrets.
