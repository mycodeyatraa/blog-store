---
id: "post-788"
title: "Shadow DOM Automation with C# in Playwright"
slug: "shadow-dom-automation-with-csharp-playwright"
date: "19-Feb-2026"
author: "pankaj-kumar"
series: "playwright-csharp-core-ui"
seriesOrder: 8
topic: "8. Shadow DOM Automation with C#"
tags: ["Playwright", "C#", "Shadow DOM", "Web Components", "DOM"]
category: "Automation Testing"
categories: ["Automation Testing", "Playwright C#", "Shadow DOM"]
excerpt: "Automate open and closed Shadow DOM web components effortlessly in Playwright C# without custom JavaScript piercing."
readTime: "8 min read"
---

# Shadow DOM Automation with C# in Playwright

Playwright C# pierces Shadow DOM trees by default across all locator selectors, allowing seamless interaction with modern Web Components.

---

## 1. Shadow DOM Piercing Architecture

```
+---------------------------------------------------------------------------------+
| Light DOM Tree                                                                  |
|   <custom-element>                                                              |
|      #shadow-root (open)                                                        |
|         <input id="shadow-input" />  <--- Playwright CSS / Role Pierces Here!   |
|   </custom-element>                                                             |
+---------------------------------------------------------------------------------+
```

---

## 2. Playwright C# Shadow DOM Test

```csharp
// ShadowDomTest.cs
using Microsoft.Playwright;
using Microsoft.Playwright.NUnit;
using NUnit.Framework;
using System.Threading.Tasks;
 
namespace MyCodeYatra.PlaywrightTests;
 
[TestFixture]
public class ShadowDomTest : PageTest
{
    [Test]
    public async Task AutomateShadowDomElements()
    {
        await Page.GotoAsync("https://mycodeyatra.com/shadow-dom");
 
        // Standard CSS locator pierces open Shadow DOM automatically
        var shadowInput = Page.Locator("custom-user-card #user-email");
        await shadowInput.FillAsync("shadow_user@mycodeyatra.com");
 
        var submitBtn = Page.GetByRole(AriaRole.Button, new() { Name = "Save Profile" });
        await submitBtn.ClickAsync();
 
        await Expect(Page.Locator("custom-user-card .status-badge")).ToHaveTextAsync("Saved");
    }
}
```

---

## 3. Enterprise Best Practices & Comparison Summary

- **Automatic Piercing**: Standard Playwright locators automatically pierce open Shadow Roots without requiring `/shadow-root` CSS pseudo-selectors.
- **Role Locators in Web Components**: Use `GetByRole()` to target accessibility roles defined inside Web Components natively.
