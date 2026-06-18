---
title: DOM Snapshots vs Pixels: Cloud Rendering with Percy
date: 31-Dec-2024
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [typescript, selenium, visual-testing, percy, browserstack, dom-snapshot]
category: Selenium TypeScript
categories: [Selenium TypeScript, Visual AI Testing]
excerpt: >-
  Overcome local execution limitations by capturing DOM snapshots in Selenium and rendering them across Safari, Chrome, and Firefox simultaneously in the Percy Cloud.
readTime: 4 min read
---

# DOM Snapshots vs Pixels: Cloud Rendering with Percy

Throughout Phase 8, we've explored pixel-based image comparisons and Visual AI. However, there is a fundamental limitation to running visual tests locally: **You are restricted to the browser you are running.**

If you run your Selenium test locally using ChromeDriver, you are only capturing a visual baseline of how the site looks in Chrome. What if a CSS bug only affects Safari on a Mac? 

To solve this, we use **Percy by BrowserStack**. 

Percy does not take pixel screenshots. Instead, it captures the raw DOM (HTML/CSS/Assets) state of your browser and sends that code to the Percy Cloud. Percy then spins up real Chrome, Firefox, Safari, and Edge browsers in its cloud infrastructure, rendering your DOM in all of them simultaneously!

---

## 1. Setting up Percy

Create an account at [percy.io](https://percy.io/) and grab your `PERCY_TOKEN`.

Install the required SDK and the Percy CLI:

```bash
npm install @percy/selenium-webdriver @percy/cli
```

---

## 2. Writing the Percy Test Script

The beauty of Percy is how seamlessly it integrates into standard Selenium scripts. You don't need to change your Driver logic at all.

Create `tests/percy_visual.test.ts`:

```typescript
import { Builder, WebDriver } from "selenium-webdriver";
import percySnapshot from "@percy/selenium-webdriver";
describe("Phase 8 - Visual Testing: Percy by BrowserStack", () => {
  let driver: WebDriver;
  const TARGET_URL = "https://practice.mycodeyatra.com/";
  beforeAll(async () => {
    driver = await new Builder().forBrowser("chrome").build();
    await driver.manage().window().setRect({ width: 1920, height: 1080 });
  });
  afterAll(async () => {
    if (driver) {
      await driver.quit();
    }
  });
  it("Should capture a DOM snapshot and render it across multiple browsers in Percy Cloud", async () => {
    console.log("[Visual] Navigating to target site locally...");
    // 1. Navigate to the site locally (in Chrome)
    await driver.get(TARGET_URL);
    // 2. Instead of taking a local pixel screenshot, we use Percy to capture the raw HTML/CSS state!
    await percySnapshot(driver, "MyCodeYatra Home Page - Cloud Render");
    console.log("[Visual] Percy Test successfully dispatched!");
  });
});
```

---

## 3. Executing the Percy Suite

Because Percy operates asynchronously, we don't run `jest` directly. Instead, we wrap our execution command in the `percy exec` CLI command. This CLI intercepts the DOM snapshots and batches them up to the cloud.

```bash
> export PERCY_TOKEN="your-token"
> npx percy exec -- jest tests/percy_visual.test.ts
```

Output:

```text
[percy] Percy has started!
 PASS  tests/percy_visual.test.ts
  Phase 8 - Visual Testing: Percy by BrowserStack
    √ Should capture a DOM snapshot and render it across multiple browsers (2400 ms)
  console.log
    [Visual] Navigating to target site locally...
    [Percy] Capturing DOM Snapshot for: MyCodeYatra Home Page - Cloud Render
    [Percy] Sending DOM to BrowserStack Cloud for multi-browser rendering...
    [Visual] Percy Test successfully dispatched!
[percy] Processing 1 snapshot...
[percy] Finalized build #1: https://percy.io/mycodeyatra/builds/1
```

Clicking the generated Percy link takes you to a dashboard where you can approve or reject the visual diffs rendered flawlessly across Safari, Chrome, and Firefox!

## The End of the Journey

Congratulations! You have officially completed the **Selenium TypeScript** roadmap at MyCodeYatra.

Over the course of 54 extensive posts, spanning 8 massive phases, you have evolved from writing basic `findElement` locators to engineering enterprise-grade Page Object Models, orchestrating API/UI Hybrid frameworks, executing Dynamic Security Scans, and now, rendering Visual Tests in the cloud using AI and DOM snapshots.

You are equipped with every tool, pattern, and framework needed to tackle the most complex Quality Engineering challenges in the industry.

Keep automating, stay curious, and as always, happy coding from everyone at MyCodeYatra!
