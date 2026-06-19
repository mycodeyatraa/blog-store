---
title: Breaking out of Java: Scaling Scans with the @axe-core/cli Tooling
date: 19-Jul-2026
lastUpdated: 19-Jul-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [java, selenium, accessibility, a11y, axe-core, nodejs, cli, test-automation]
category: Selenium Java
categories: [Selenium Java, Accessibility Testing]
excerpt: >-
  You don't always need Selenium for accessibility testing. Learn how to use the NodeJS @axe-core/cli to run lightning-fast headless WCAG scans from your terminal and export results to CSV.
readTime: 6 min read
---

# Breaking out of Java: Scaling Scans with the @axe-core/cli Tooling

Throughout this series, we have successfully integrated the `axe-selenium` Java SDK directly into our UI automation framework. This is the perfect approach for testing complex, multi-step user workflows like logging in or checking out a shopping cart.

However, what if you just want to run a lightning-fast WCAG compliance scan across 50 static blog pages on your website?

Spinning up 50 instances of ChromeDriver via Java is complete overkill. For rapid, static page scanning, Deque provides a headless NodeJS command-line tool known as `@axe-core/cli`.

In this tutorial, we will learn how to bypass Selenium entirely and trigger headless accessibility scans directly from your terminal or CI/CD pipeline!

---

## 1. Installing the Axe CLI

Because the Axe CLI is built on NodeJS, you must have Node installed on your machine.

Open your terminal and install the tool globally via `npm`:

```bash
npm install -g @axe-core/cli
```

Verify the installation by checking the version:

```bash
axe --version
```

---

## 2. Running a Basic CLI Scan

The Axe CLI uses Puppeteer (a headless Chrome browser) under the hood. To scan a webpage, simply pass the URL to the `axe` command:

```bash
axe https://practice.mycodeyatra.com/login
```

Within seconds, the CLI will output a beautiful, color-coded report directly into your terminal:

```text
Running axe-core 4.9.1 in headless chrome
Testing https://practice.mycodeyatra.com/login...
2 Accessibility Violations Found:
1. color-contrast (Ensures the contrast between foreground and background colors meets WCAG 2 AA contrast ratio thresholds)
   Target: #submit-btn
   Impact: serious
   Help: https://dequeuniversity.com/rules/axe/4.9/color-contrast
2. html-has-lang (Ensures every HTML document has a lang attribute)
   Target: html
   Impact: serious
   Help: https://dequeuniversity.com/rules/axe/4.9/html-has-lang
```

---

## 3. Customizing the CLI Execution

The CLI is incredibly flexible and supports dozens of flags to customize your scans.

**Scan multiple URLs simultaneously:**

```bash
axe https://practice.mycodeyatra.com/ https://practice.mycodeyatra.com/about
```

**Only run specific WCAG rules (e.g., only check Color Contrast):**

```bash
axe https://practice.mycodeyatra.com/ --rules color-contrast
```

**Run against a specific WCAG Standard (e.g., WCAG 2.1 AA):**

```bash
axe https://practice.mycodeyatra.com/ --tags wcag21aa
```

**Bypass Authentication (Inject Cookies):**
If you need to scan a dashboard that requires a login, you can pass a valid session cookie directly via the CLI!

```bash
axe https://practice.mycodeyatra.com/dashboard --cookie "session_id=123xyz"
```

---

## 4. Generating Artifacts for CI/CD

Just like our ExtentReports implementation in Java, the terminal output of the CLI is useless for enterprise auditing. You need to export the results.

The CLI allows you to instantly dump the raw JSON or output a consolidated CSV file:

**Export to JSON:**

```bash
axe https://practice.mycodeyatra.com/ --save report.json
```

**Export to CSV (Perfect for Excel/Product Managers):**

```bash
axe https://practice.mycodeyatra.com/ --save report.csv
```

If you are running this in a GitHub Actions or Jenkins pipeline, the CLI will automatically exit with a **Code 1** (failing the pipeline) if any violations are found. If you want the scan to run but *not* fail the build, you can use the `--exit` flag:

```bash
axe https://practice.mycodeyatra.com/ --exit 0
```

## Conclusion

The `@axe-core/cli` is the ultimate tool for "Shift-Left" accessibility testing. While Selenium is required for complex, stateful workflows, the CLI is unbeatable for rapidly scanning dozens of static URLs and instantly exporting the compliance data to CSV.

By combining the Axe Java SDK in your functional tests with the Axe CLI in your CI/CD pipelines, you achieve 100% test coverage for WCAG compliance.

In our final tutorial of the Accessibility Testing Phase, we will discuss **Enterprise Accessibility Strategy**, exploring how to convince management to care about a11y, and how to gradually introduce these tools to a legacy codebase without breaking the build!
