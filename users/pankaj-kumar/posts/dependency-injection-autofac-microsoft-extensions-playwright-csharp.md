---
id: "post-802"
title: "Dependency Injection with Autofac & Microsoft.Extensions.DependencyInjection in Playwright C#"
slug: "dependency-injection-autofac-microsoft-extensions-playwright-csharp"
date: "05-Mar-2026"
author: "pankaj-kumar"
series: "playwright-csharp-test-design"
seriesOrder: 11
topic: "11. Dependency Injection with Autofac & Microsoft.Extensions.DependencyInjection"
tags: ["Playwright", "C#", "Dependency Injection", "Autofac", "DI"]
category: "Automation Testing"
categories: ["Automation Testing", "Playwright C#", "Dependency Injection"]
excerpt: "Inject Page Objects, configuration, and services using Microsoft.Extensions.DependencyInjection in Playwright C#."
readTime: "8 min read"
---

# Dependency Injection with Autofac & Microsoft.Extensions.DependencyInjection in Playwright C#

Dependency Injection (DI) manages object creation and lifecycles automatically, reducing boilerplate code for initializing Page Objects and services.

---

## 1. Dependency Injection Flow

```
+------------------------------------+
| ServiceProvider (DI Container)     |
+------------------------------------+
                  |
                  v Resolves LoginPage with IPage injected
+------------------------------------+
| LoginPage(IPage page) Instance     |
+------------------------------------+
```

---

## 2. DI Container Setup & Registration

```csharp
// DiSetup.cs
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Playwright;
using MyCodeYatra.Pages;
 
namespace MyCodeYatra.Di;
 
public static class DiSetup
{
    public static ServiceProvider ConfigureServices(IPage page)
    {
        var services = new ServiceCollection();
        
        services.AddSingleton(page);
        services.AddTransient<LoginPage>();
        
        return services.BuildServiceProvider();
    }
}
```

---

## 3. Playwright Test with Injected Services

```csharp
// DiTest.cs
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Playwright.NUnit;
using MyCodeYatra.Di;
using MyCodeYatra.Pages;
using NUnit.Framework;
using System.Threading.Tasks;
 
namespace MyCodeYatra.PlaywrightTests;
 
[TestFixture]
public class DiTest : PageTest
{
    [Test]
    public async Task TestPageObjectWithDependencyInjection()
    {
        var container = DiSetup.ConfigureServices(Page);
        var loginPage = container.GetRequiredService<LoginPage>();
 
        await loginPage.NavigateAsync();
        await loginPage.PerformLoginAsync("qa_user", "Secret123!");
 
        await Expect(Page.Locator(".welcome-banner")).ToBeVisibleAsync();
    }
}
```

---

## 4. Enterprise Best Practices & Comparison Summary

- **Transient Page Objects**: Register Page Objects as `Transient` services so fresh instances are constructed per injection.
- **Service Decoupling**: Inject interfaces (`IServices`) instead of concrete classes for easy test double mocking.
