---
title: A Picture is Worth a Thousand Tests: Allure Reporting Basics
date: 26-Jan-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [typescript, selenium, reporting, allure, dashboards, automation]
category: Selenium TypeScript
categories: [Selenium TypeScript, Reporting]
excerpt: >-
  Transform ugly terminal output into beautiful, executive-friendly web dashboards by integrating the Allure Reporting framework into your Selenium TypeScript architecture.
readTime: 4 min read
---

# A Picture is Worth a Thousand Tests: Allure Reporting Basics

Over the past 11 phases, we have built a fully functional enterprise test automation pipeline. It runs cross-browser tests in Docker containers via GitHub Actions on a nightly schedule. 

But there is a major problem with standard test runners (like Jest or Cucumber): their output is usually just a raw stream of text in the terminal.

If a test fails, you get a stack trace. 

Product Managers, Business Analysts, and QA Directors do not want to read stack traces. They want pie charts, graphs, step-by-step logs, and embedded screenshots. 

They want **Allure**.

---

## 1. What is Allure?

Allure Framework is an open-source, multi-language test reporting tool. It is designed to create visually stunning, highly interactive HTML reports that everyone on the team can understand.

It doesn't replace your test runner; it *augments* it. You plug Allure into Cucumber (or Jest, or Mocha), and as your tests run, Allure silently collects data (steps, screenshots, durations). When the run is complete, Allure compiles that data into a beautiful web application.

---

## 2. Installing Allure for Cucumber-TS

First, we need to install the Allure command-line tool (to generate the report) and the specific Allure Cucumber plugin.

```bash
npm install --save-dev allure-commandline allure-cucumberjs
```

---

## 3. Configuring Cucumber to use Allure

By default, Cucumber uses its own built-in HTML reporter or logs directly to the console. We need to tell Cucumber to send its data to Allure instead.

To do this, we create a specific "Reporter" configuration file. In your project root, create a file named `reporter.ts`:

```typescript
import { AllureRuntime, CucumberJSAllureFormatter } from "allure-cucumberjs";
export default class Reporter extends CucumberJSAllureFormatter {
    constructor(options: any) {
        super(
            options,
            new AllureRuntime({ resultsDir: "./allure-results" }),
            {}
        );
    }
}
```

Next, update your `cucumber.js` profile to explicitly point to this new reporter:

```javascript
module.exports = {
  default: `--require-module ts-node/register --require tests/bdd/**/*.ts --format ./reporter.ts`
};
```

---

## 4. Running the Tests and Generating the Report

When you execute your tests using `npm run test:bdd`, Allure will automatically intercept the results and dump raw JSON data into a new directory named `./allure-results`.

However, raw JSON files are not a human-readable dashboard. 

To convert those JSON files into a beautiful HTML website, we must run the Allure command-line tool. Add these scripts to your `package.json`:

```json
  "scripts": {
    "test:bdd": "cucumber-js",
    "report:generate": "allure generate ./allure-results --clean -o ./allure-report",
    "report:open": "allure open ./allure-report"
  }
```

### The Execution Flow:
1. Run `npm run test:bdd`. (Tests execute; `./allure-results` is populated).
2. Run `npm run report:generate`. (Allure converts the JSON into HTML in `./allure-report`).
3. Run `npm run report:open`. (Allure boots up a local web server to display the dashboard in your browser!).

## Conclusion

With a simple plugin installation, we have transformed our ugly terminal output into a sleek, professional web dashboard featuring interactive graphs, timelines, and categorizations.

But Allure can do much more than just log "Pass" or "Fail". 

In our next tutorial, **Advanced Allure**, we will learn how to embed high-resolution screenshots directly into the report, attach custom logs, and tag tests with Criticality severity levels!
