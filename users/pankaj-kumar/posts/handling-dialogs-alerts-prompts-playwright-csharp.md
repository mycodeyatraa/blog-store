---
id: "post-783"
title: "Handling Dialogs, Alerts & Prompts in Playwright C#"
slug: "handling-dialogs-alerts-prompts-playwright-csharp"
date: "14-Feb-2026"
author: "pankaj-kumar"
series: "playwright-csharp-core-ui"
seriesOrder: 3
topic: "3. Handling Dialogs, Alerts & Prompts"
tags: ["Playwright", "C#", "Dialogs", "Alerts", "Prompts"]
category: "Automation Testing"
categories: ["Automation Testing", "Playwright C#", "Dialogs"]
excerpt: "Handle browser dialogs, JavaScript alerts, confirms, and prompts automatically using Playwright C# event listeners."
readTime: "8 min read"
---

# Handling Dialogs, Alerts & Prompts in Playwright C#

Playwright automatically dismisses JavaScript dialogs by default to prevent tests from hanging. Custom event listeners allow accepting, rejecting, or providing input to prompts.

---

## 1. Dialog Event Listener Architecture

```
+------------------------------------+
| Attach Dialog Listener             |
| Page.Dialog += async (sender, d)   |
+------------------------------------+
                  |
                  v Trigger JS Alert/Confirm/Prompt
+------------------------------------+
| Action: AcceptAsync() / Dismiss()  |
+------------------------------------+
```

---

## 2. Playwright C# Dialog Handling Test

```csharp
// DialogsTest.cs
using Microsoft.Playwright;
using Microsoft.Playwright.NUnit;
using NUnit.Framework;
using System.Threading.Tasks;
 
namespace MyCodeYatra.PlaywrightTests;
 
[TestFixture]
public class DialogsTest : PageTest
{
    [Test]
    public async Task HandleJavaScriptConfirmDialog()
    {
        await Page.GotoAsync("https://mycodeyatra.com/dialogs");
 
        // Attach event handler before triggering dialog
        Page.Dialog += async (_, dialog) =>
        {
            Assert.That(dialog.Message, Is.EqualTo("Are you sure you want to delete this record?"));
            await dialog.AcceptAsync();
        };
 
        await Page.GetByRole(AriaRole.Button, new() { Name = "Delete Item" }).ClickAsync();
        await Expect(Page.Locator("#result-msg")).ToHaveTextAsync("Item Deleted Successfully");
    }
 
    [Test]
    public async Task HandleJavaScriptPromptDialog()
    {
        await Page.GotoAsync("https://mycodeyatra.com/dialogs");
 
        Page.Dialog += async (_, dialog) =>
        {
            await dialog.AcceptAsync("AutomationUser");
        };
 
        await Page.GetByRole(AriaRole.Button, new() { Name = "Enter Name" }).ClickAsync();
        await Expect(Page.Locator("#greeting")).ToHaveTextAsync("Hello, AutomationUser!");
    }
}
```

---

## 3. Enterprise Best Practices & Comparison Summary

- **Event Registration**: Register the `Page.Dialog` event listener *before* performing the action that triggers the popup.
- **Message Inspection**: Validate `dialog.Message` string inside the listener to confirm correct dialog activation.
