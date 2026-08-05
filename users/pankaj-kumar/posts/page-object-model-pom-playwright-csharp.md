---
id: "post-793"
title: "Page Object Model (POM) in Playwright C#"
slug: "page-object-model-pom-playwright-csharp"
date: "24-Feb-2026"
author: "pankaj-kumar"
series: "playwright-csharp-test-design"
seriesOrder: 2
topic: "2. Page Object Model"
tags: ["Playwright", "C#", "POM", "Page Objects", "Architecture"]
category: "Automation Testing"
categories: ["Automation Testing", "Playwright C#", "Page Object Model"]
excerpt: "Design clean Page Object Model (POM) classes in Playwright C# using encapsulation and locator properties."
readTime: "8 min read"
---

# Page Object Model (POM) in Playwright C#

The Page Object Model (POM) design pattern encapsulates HTML page elements and user interactions inside clean C# class representations.

---

## 1. Page Object Model Architecture

```
+------------------------------------+       +------------------------------------+
| Test Suite (LoginPageTest.cs)      | ----> | Page Object (LoginPage.cs)         |
| (Calls page.LoginAsync())          |       | (Encapsulates Locators & Actions)  |
+------------------------------------+       +------------------------------------+
```

---

## 2. C# Page Object Class

```csharp
// LoginPage.cs
using Microsoft.Playwright;
using System.Threading.Tasks;
 
namespace MyCodeYatra.Pages;
 
public class LoginPage
{
    private readonly IPage _page;
    private readonly ILocator _usernameInput;
    private readonly ILocator _passwordInput;
    private readonly ILocator _loginButton;
 
    public LoginPage(IPage page)
    {
        _page = page;
        _usernameInput = page.GetByLabel("Username");
        _passwordInput = page.GetByLabel("Password");
        _loginButton = page.GetByRole(AriaRole.Button, new() { Name = "Sign In" });
    }
 
    public async Task NavigateAsync()
    {
        await _page.GotoAsync("https://mycodeyatra.com/login");
    }
 
    public async Task PerformLoginAsync(string username, string password)
    {
        await _usernameInput.FillAsync(username);
        await _passwordInput.FillAsync(password);
        await _loginButton.ClickAsync();
    }
}
```

---

## 3. Playwright POM Execution Test

```csharp
// PomTest.cs
using Microsoft.Playwright.NUnit;
using MyCodeYatra.Pages;
using NUnit.Framework;
using System.Threading.Tasks;
 
namespace MyCodeYatra.PlaywrightTests;
 
[TestFixture]
public class PomTest : PageTest
{
    [Test]
    public async Task VerifyLoginUsingPageObject()
    {
        var loginPage = new LoginPage(Page);
        await loginPage.NavigateAsync();
        await loginPage.PerformLoginAsync("qa_admin", "AdminPass123!");
 
        await Expect(Page.Locator(".dashboard-welcome")).ToBeVisibleAsync();
    }
}
```

---

## 4. Enterprise Best Practices & Comparison Summary

- **ILocator Properties**: Store `ILocator` objects as private fields inside Page Objects to enable lazy auto-wait evaluation.
- **Fluent Method Chaining**: Return `this` or subsequent Page Object instances from navigation methods for readable test syntax.
