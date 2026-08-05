---
id: "post-795"
title: "Builder Pattern for Test Data Generation in Playwright C#"
slug: "builder-pattern-test-data-generation-playwright-csharp"
date: "26-Feb-2026"
author: "pankaj-kumar"
series: "playwright-csharp-test-design"
seriesOrder: 4
topic: "4. Builder Pattern"
tags: ["Playwright", "C#", "Builder Pattern", "Test Data", "Fluent"]
category: "Automation Testing"
categories: ["Automation Testing", "Playwright C#", "Design Patterns"]
excerpt: "Construct complex, customizable test data models in C# using Fluent Builder patterns for Playwright test suites."
readTime: "8 min read"
---

# Builder Pattern for Test Data Generation in Playwright C#

The Builder pattern provides a fluent interface to construct complex domain objects with sensible defaults for test scenarios.

---

## 1. Builder Pattern Flow

```
+---------------------------------------------------------------------------------+
| UserBuilder.Create().WithEmail("a@b.com").WithRole("ADMIN").Build()             |
+---------------------------------------------------------------------------------+
```

---

## 2. User Data Builder Implementation

```csharp
// UserBuilder.cs
using System;
 
namespace MyCodeYatra.Builders;
 
public class UserData
{
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public string Email { get; set; }
    public string Role { get; set; }
}
 
public class UserBuilder
{
    private readonly UserData _user = new()
    {
        FirstName = "DefaultFirst",
        LastName = "DefaultLast",
        Email = $"user_{Guid.NewGuid():N}@mycodeyatra.com",
        Role = "USER"
    };
 
    public static UserBuilder Create() => new();
 
    public UserBuilder WithRole(string role)
    {
        _user.Role = role;
        return this;
    }
 
    public UserBuilder WithEmail(string email)
    {
        _user.Email = email;
        return this;
    }
 
    public UserData Build() => _user;
}
```

---

## 3. Playwright Builder Pattern Test

```csharp
// BuilderTest.cs
using Microsoft.Playwright.NUnit;
using MyCodeYatra.Builders;
using NUnit.Framework;
using System.Threading.Tasks;
 
namespace MyCodeYatra.PlaywrightTests;
 
[TestFixture]
public class BuilderTest : PageTest
{
    [Test]
    public async Task TestUserRegistrationWithBuilderData()
    {
        var adminUser = UserBuilder.Create()
            .WithRole("ADMIN")
            .WithEmail("admin_qa@mycodeyatra.com")
            .Build();
 
        await Page.GotoAsync("https://mycodeyatra.com/register");
        await Page.GetByLabel("Email").FillAsync(adminUser.Email);
        await Page.GetByRole(AriaRole.Button, new() { Name = "Register" }).ClickAsync();
 
        await Expect(Page.Locator(".user-role")).ToHaveTextAsync("ADMIN");
    }
}
```

---

## 4. Enterprise Best Practices & Comparison Summary

- **Sensible Defaults**: Initialize default field values inside the builder so tests only specify fields relevant to their scenario.
- **Randomized Identifiers**: Generate unique GUID emails by default to guarantee test data isolation across parallel builds.
