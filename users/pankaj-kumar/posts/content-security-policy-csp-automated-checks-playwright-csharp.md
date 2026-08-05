---
id: "post-818"
title: "Content Security Policy (CSP) Automated Checks in Playwright C#"
slug: "content-security-policy-csp-automated-checks-playwright-csharp"
date: "21-Mar-2026"
author: "pankaj-kumar"
series: "playwright-csharp-security-testing"
seriesOrder: 2
topic: "2. Content Security Policy (CSP) Automated Checks"
tags: ["Playwright", "C#", "CSP", "Security", "XSS"]
category: "Automation Testing"
categories: ["Automation Testing", "Playwright C#", "CSP Testing"]
excerpt: "Validate Content Security Policy (CSP) directives and catch script injection violations using Playwright C# page console listeners."
readTime: "8 min read"
---

# Content Security Policy (CSP) Automated Checks in Playwright C#

Content Security Policy (CSP) mitigates Cross-Site Scripting (XSS) attacks by restricting script origins and inline script execution.

---

## 1. CSP Violation Monitoring Architecture

```
+------------------------------------+
| Page.Console += (msg => ...)       |
+------------------------------------+
                  |
                  v Intercepts CSP Violation Errors
+------------------------------------+
| Assert Zero CSP Console Errors     |
+------------------------------------+
```

---

## 2. Playwright C# CSP Audit Test

```csharp
// CspAuditTest.cs
using Microsoft.Playwright;
using Microsoft.Playwright.NUnit;
using NUnit.Framework;
using System.Collections.Generic;
using System.Threading.Tasks;
 
namespace MyCodeYatra.PlaywrightTests;
 
[TestFixture]
public class CspAuditTest : PageTest
{
    [Test]
    public async Task AuditCspDirectivesAndViolations()
    {
        var cspViolations = new List<string>();
 
        // Listen for browser console CSP error logs
        Page.Console += (_, msg) =>
        {
            if (msg.Type == "error" && msg.Text.Contains("Content Security Policy"))
            {
                cspViolations.Add(msg.Text);
            }
        };
 
        var response = await Page.GotoAsync("https://mycodeyatra.com");
        var headers = response.Headers;
 
        Assert.That(headers, Contains.Key("content-security-policy"), "CSP Header must be configured");
        Assert.That(cspViolations.Count, Is.EqualTo(0), "No CSP violation errors should be logged in console");
    }
}
```

---

## 3. Enterprise Best Practices & Comparison Summary

- **Restrict Script Sources**: Ensure CSP directives include `script-src 'self'` and disallow `'unsafe-inline'`.
- **Console Listener Interception**: Listen to `Page.Console` to capture CSP violation errors automatically.
