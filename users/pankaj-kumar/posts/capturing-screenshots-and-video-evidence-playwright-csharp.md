---
id: "post-780"
title: "Capturing Screenshots & Video Evidence in Playwright C#"
slug: "capturing-screenshots-and-video-evidence-playwright-csharp"
date: "11-Feb-2026"
author: "pankaj-kumar"
series: "playwright-csharp-foundations"
seriesOrder: 10
topic: "10. Screenshots & Videos"
tags: ["Playwright", "C#", "Screenshots", "Video", "Evidence"]
category: "Automation Testing"
categories: ["Automation Testing", "Playwright C#", "Evidence Collection"]
excerpt: "Capture full-page screenshots, element screenshots, and video recordings in Playwright C# for test audit trails."
readTime: "8 min read"
---

# Capturing Screenshots & Video Evidence in Playwright C#

Collecting visual evidence (PNG screenshots and WebM video recordings) is essential for diagnosing transient UI bugs in CI/CD pipelines.

---

## 1. Visual Evidence Capture Flow

```
+------------------------------------+
| NewContextAsync(RecordVideoDir)    |
+------------------------------------+
                  |
                  v Execute Test Actions
+------------------------------------+
| Page.ScreenshotAsync()             |
+------------------------------------+
                  |
                  v Save PNG & Video Artifacts
+------------------------------------+
| target/screenshots/ & target/videos|
+------------------------------------+
```

---

## 2. Playwright C# Evidence Collection Test

```csharp
// ScreenshotVideoTest.cs
using Microsoft.Playwright;
using NUnit.Framework;
using System.IO;
using System.Threading.Tasks;
 
[TestFixture]
public class ScreenshotVideoTest
{
    [Test]
    public async Task CaptureScreenshotAndVideo()
    {
        using var playwright = await Playwright.CreateAsync();
        await using var browser = await playwright.Chromium.LaunchAsync(new BrowserTypeLaunchOptions { Headless = true });
 
        // Enable Video Recording
        var context = await browser.NewContextAsync(new BrowserNewContextOptions
        {
            RecordVideoDir = "target/videos/"
        });
 
        var page = await context.NewPageAsync();
        await page.GotoAsync("https://mycodeyatra.com");
 
        // 1. Full-Page PNG Screenshot
        Directory.CreateDirectory("target/screenshots");
        await page.ScreenshotAsync(new PageScreenshotOptions
        {
            Path = "target/screenshots/full_homepage.png",
            FullPage = true
        });
 
        // 2. Specific Element PNG Screenshot
        var logo = page.Locator(".brand-logo");
        if (await logo.IsVisibleAsync())
        {
            await logo.ScreenshotAsync(new LocatorScreenshotOptions
            {
                Path = "target/screenshots/logo.png"
            });
        }
 
        await context.CloseAsync(); // Closes context to finalize video writing
        Assert.That(File.Exists("target/screenshots/full_homepage.png"), Is.True);
    }
}
```

---

## 3. Enterprise Best Practices & Comparison Summary

- **Conditional Screenshots**: Capture screenshots on failure inside NUnit `TearDown` hooks to save disk space.
- **Context Finalization**: Always call `await context.CloseAsync()` before asserting video file existence to ensure the WebM file writer finishes.
