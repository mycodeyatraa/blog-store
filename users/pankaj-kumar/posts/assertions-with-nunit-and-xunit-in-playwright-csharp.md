---
id: "post-777"
title: "Assertions with NUnit and xUnit in Playwright C#"
slug: "assertions-with-nunit-and-xunit-in-playwright-csharp"
date: "08-Feb-2026"
author: "pankaj-kumar"
series: "playwright-csharp-foundations"
seriesOrder: 7
topic: "7. Assertions with NUnit / xUnit"
tags: ["Playwright", "C#", "Assertions", "NUnit", "xUnit"]
category: "Automation Testing"
categories: ["Automation Testing", "Playwright C#", "Assertions"]
excerpt: "Master web-first retrying assertions in Playwright C# using `Expect(Locator)` for resilient test suites."
readTime: "8 min read"
---

# Assertions with NUnit and xUnit in Playwright C#

Web-first assertions automatically wait and retry until expected DOM conditions are met, eliminating manual sleep delays and flakiness.

---

## 1. Web-First vs Standard Assertions Architecture

```
+---------------------------------------------------------------------------------+
| Standard Assertion (Assert.True(page.IsVisible("#btn")))                        |
| -> Evaluates DOM instantaneously. Fails if element renders 10ms later!          |
+---------------------------------------------------------------------------------+
                                       vs
+---------------------------------------------------------------------------------+
| Web-First Retrying Assertion (await Expect(Locator).ToBeVisibleAsync())        |
| -> Automatically polls DOM for up to 5 seconds until condition passes!         |
+---------------------------------------------------------------------------------+
```

---

## 2. Playwright Web-First Assertions Test

```csharp
// AssertionsTest.cs
using Microsoft.Playwright;
using Microsoft.Playwright.NUnit;
using NUnit.Framework;
using System.Threading.Tasks;
 
namespace MyCodeYatra.PlaywrightTests;
 
[TestFixture]
public class AssertionsTest : PageTest
{
    [Test]
    public async Task VerifyWebFirstAssertions()
    {
        await Page.GotoAsync("https://mycodeyatra.com");
 
        // 1. Title Assertion
        await Expect(Page).ToHaveTitleAsync("MyCodeYatra");
 
        // 2. Locator Visibility Assertion
        var header = Page.GetByRole(AriaRole.Heading, new() { Name = "Welcome to MyCodeYatra" });
        await Expect(header).ToBeVisibleAsync();
 
        // 3. Locator Attribute & Text Assertions
        var searchBtn = Page.GetByRole(AriaRole.Button, new() { Name = "Search" });
        await Expect(searchBtn).ToBeEnabledAsync();
        await Expect(searchBtn).ToHaveAttributeAsync("type", "submit");
    }
}
```

---

## 3. Enterprise Best Practices & Comparison Summary

- **Always Use Web-First Assertions**: Never pass `page.IsVisibleAsync()` inside `Assert.IsTrue()`. Always use `await Expect(locator).ToBeVisibleAsync()`.
- **Custom Timeouts**: Pass custom timeout parameters (`await Expect(locator).ToBeVisibleAsync(new() { Timeout = 10000 })`) for long-running UI updates.
