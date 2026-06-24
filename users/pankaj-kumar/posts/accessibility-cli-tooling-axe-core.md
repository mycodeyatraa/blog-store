---
title: Zero-Code Accessibility Audits with the Axe-Core CLI Tooling
date: 14-Apr-2025
lastUpdated: 14-Apr-2025
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: ["playwright", "typescript", "axe-core", "accessibility", "cli"]
category: UI Automation
categories: ["UI Automation", "Playwright", "TypeScript", "Accessibility", "CLI Tooling"]
excerpt: >-
  Learn how to execute instant, ad-hoc WCAG accessibility scans on any public URL directly from your terminal using axe-core/cli.
readTime: 3 min read
---

While integrating accessibility checks directly into your Playwright UI automation framework is highly recommended, sometimes developers and QA testers just need a fast, ad-hoc way to audit a page without writing any code.

In this tutorial, we will explore `@axe-core/cli`, a command-line interface that allows you to instantly scan any public URL for WCAG accessibility violations.

### Installing the Axe-Core CLI

You can install the CLI globally on your machine, or save it as a dev dependency inside your repository.

```bash
# Install globally
npm install -g @axe-core/cli
 
# OR install locally
npm install -D @axe-core/cli
```

### Running an Ad-Hoc Accessibility Scan

To scan a website, simply pass the target URL to the `axe` command:

```bash
npx axe https://practice.mycodeyatra.com/
```

### Understanding the CLI Output

The CLI leverages a headless browser to render the DOM and execute the rules engine. It will print a highly readable summary of all violations directly to your terminal screen.

For example, when scanning our practice application, it quickly isolates the identical `color-contrast` issues our Playwright tests previously discovered:

```text
Running axe-core 4.10.0 in chrome-headless
 
Target: https://practice.mycodeyatra.com/
 
2 violations found:
 
1. color-contrast: Elements must have sufficient color contrast
   - Impact: serious
   - Help: https://dequeuniversity.com/rules/axe/4.10/color-contrast
   - Target: .card.hover-card.glass:nth-child(42) > div:nth-child(2) > button
     Fix any of the following:
       Element has insufficient color contrast of 4.23 (foreground color: #8b5cf6, background color: #ffffff, font size: 9.6pt (12.8px), font weight: bold). Expected contrast ratio of 4.5:1
 
   - Target: .card.hover-card.glass:nth-child(46) > div:nth-child(2) > button
     Fix any of the following:
       Element has insufficient color contrast of 3.85 (foreground color: #0288d1, background color: #ffffff, font size: 9.6pt (12.8px), font weight: bold). Expected contrast ratio of 4.5:1
```

### Saving Results to a File

If you want to save the output for later analysis or share it with your team, you can output the raw JSON payload to a file using the `--save` parameter:

```bash
npx axe https://practice.mycodeyatra.com/ --save axe-results.json
```

### Summary

The `@axe-core/cli` is an indispensable tool for rapid, zero-code accessibility audits. By running a simple terminal command, developers can get immediate feedback on color contrast, missing ARIA tags, and structural violations without needing to build and maintain a complete UI automation framework!
