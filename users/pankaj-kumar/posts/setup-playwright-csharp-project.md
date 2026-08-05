---
id: "post-773"
title: "Setting Up a Playwright C# Project"
slug: "setup-playwright-csharp-project"
date: "04-Feb-2026"
author: "pankaj-kumar"
series: "playwright-csharp-foundations"
seriesOrder: 3
topic: "3. Setup Playwright + C# Project"
tags: ["Playwright", "C#", "NuGet", ".NET", "Setup"]
category: "Automation Testing"
categories: ["Automation Testing", "Playwright C#", "Project Setup"]
excerpt: "Step-by-step guide to installing Microsoft.Playwright NuGet packages and executing `playwright.ps1 install` in .NET 8 / 9 solutions."
readTime: "8 min read"
---

# Setting Up a Playwright C# Project

Setting up Playwright in a C# environment requires initializing a .NET test project, adding the `Microsoft.Playwright.NUnit` package, and installing browser binaries.

---

## 1. CLI Commands for Project Setup

```bash
# Create a new NUnit test project
dotnet new nunit -n MyCodeYatra.PlaywrightTests
cd MyCodeYatra.PlaywrightTests
 
# Add Playwright NuGet package
dotnet add package Microsoft.Playwright.NUnit
 
# Build project to copy Playwright scripts to bin
dotnet build
 
# Install Playwright browser binaries
pwsh bin/Debug/net8.0/playwright.ps1 install
```

---

## 2. NUnit Playwright Test Base Class

```csharp
// UnitTest1.cs
using Microsoft.Playwright.NUnit;
using NUnit.Framework;
using System.Threading.Tasks;
 
namespace MyCodeYatra.PlaywrightTests;
 
[TestFixture]
public class Tests : PageTest
{
    [Test]
    public async Task HomepageHasCorrectTitle()
    {
        await Page.GotoAsync("https://mycodeyatra.com");
        await Expect(Page).ToHaveTitleAsync("MyCodeYatra");
    }
}
```

---

## 3. Enterprise Best Practices & Comparison Summary

- **`PageTest` Base Class**: Inheriting from `PageTest` in `Microsoft.Playwright.NUnit` provides automatic browser lifecycle and screenshot management.
- **Powershell Script**: Always run `playwright.ps1 install` after building the solution to ensure required browser binaries are present.
