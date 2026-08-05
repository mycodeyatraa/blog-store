---
id: "post-816"
title: "SSO & Enterprise MFA Patterns (Azure AD / Okta) in Playwright C#"
slug: "sso-enterprise-mfa-patterns-azure-ad-okta-playwright-csharp"
date: "19-Mar-2026"
author: "pankaj-kumar"
series: "playwright-csharp-api-testing"
seriesOrder: 13
topic: "13. SSO & Enterprise MFA Patterns (Azure AD / Okta)"
tags: ["Playwright", "C#", "SSO", "Azure AD", "Okta", "MFA"]
category: "Automation Testing"
categories: ["Automation Testing", "Playwright C#", "SSO & MFA"]
excerpt: "Automate Single Sign-On (SSO) login flows and handle OTP multi-factor authentication (MFA) in Playwright C#."
readTime: "8 min read"
---

# SSO & Enterprise MFA Patterns (Azure AD / Okta) in Playwright C#

Automating Single Sign-On (SSO) through Identity Providers (IdPs) like Azure AD (Entra ID) or Okta requires handling redirects and TOTP/MFA challenges.

---

## 1. Enterprise SSO & MFA Architecture

```
+----------------+      +----------------+      +-----------------+      +--------------------+
| 1. App Redirect| ---> | 2. Azure AD /  | ---> | 3. MFA TOTP     | ---> | 4. Session Token   |
| to IdP Login   |      | Okta Login     |      | Verification    |      | Returned to App    |
+----------------+      +----------------+      +-----------------+      +--------------------+
```

---

## 2. Playwright C# SSO Handling Test

```csharp
// SsoMfaTest.cs
using Microsoft.Playwright;
using Microsoft.Playwright.NUnit;
using NUnit.Framework;
using System.Threading.Tasks;
 
namespace MyCodeYatra.PlaywrightTests;
 
[TestFixture]
public class SsoMfaTest : PageTest
{
    [Test]
    public async Task HandleEnterpriseAzureAdLogin()
    {
        await Page.GotoAsync("https://mycodeyatra.com/sso/login");
 
        // 1. Handle IdP Redirect to Microsoft Azure AD
        await Page.GetByPlaceholder("Email, phone, or Skype").FillAsync("qa_admin@mycodeyatra.com");
        await Page.GetByRole(AriaRole.Button, new() { Name = "Next" }).ClickAsync();
 
        await Page.GetByPlaceholder("Password").FillAsync("EnterprisePassword123!");
        await Page.GetByRole(AriaRole.Button, new() { Name = "Sign in" }).ClickAsync();
 
        // 2. Handle "Stay signed in?" prompt
        var staySignedInBtn = Page.GetByRole(AriaRole.Button, new() { Name = "Yes" });
        if (await staySignedInBtn.IsVisibleAsync())
        {
            await staySignedInBtn.ClickAsync();
        }
 
        // 3. Verify return to application dashboard
        await Expect(Page.Locator(".sso-welcome")).ToBeVisibleAsync();
    }
}
```

---

## 3. Enterprise Best Practices & Comparison Summary

- **Service Principal Tokens**: Acquire OAuth tokens via Azure AD App Registrations directly via API to bypass MFA UI prompts in CI/CD pipelines.
- **TOTP Generation**: Use `Otplib` or `TwoStepsAuthenticator` C# libraries to generate 6-digit TOTP codes dynamically if MFA cannot be disabled in test environments.
