---
title: Scaling TypeScript UI Automation: Jest Parallel Execution
date: 17-Nov-2024
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [typescript, selenium, jest, parallel-execution, ci-cd]
category: Selenium TypeScript
categories: [Selenium TypeScript, UI Automation, Core Automation]
excerpt: >-
  Reduce your total execution time by running UI tests concurrently. Learn how to configure Jest for parallel execution and thread-safe WebDrivers.
readTime: 4 min read
---

# Scaling TypeScript UI Automation: Jest Parallel Execution

As your automation suite grows from 10 tests to 100 tests, execution time becomes a critical bottleneck. If each UI test takes 10 seconds, 100 tests will take nearly 17 minutes to complete! 

To provide rapid feedback in CI/CD pipelines, we must execute our tests concurrently. 

In this tutorial, we will learn how to configure **Jest** to run our Selenium WebDriver tests in parallel, drastically reducing our total execution time.

---

## 1. How Jest Parallelism Works

By default, if you run `jest`, it will often run tests sequentially in a single process to save memory, or it may attempt to spawn a worker for every CPU core available.

For UI automation, spawning 16 Chrome browsers simultaneously on a laptop will crash the OS! We need to explicitly cap the maximum number of concurrent browsers.

Jest allows us to control this via the `maxWorkers` configuration.

---

## 2. Updating jest.config.js

Open your `jest.config.js` file at the root of your project and add the `maxWorkers` property. We will also increase `testTimeout` to 60 seconds, as parallel browsers compete for CPU resources and take slightly longer to render pages.

```javascript
/** @type {import('ts-jest').JestConfigWithTsJest} */
module.exports = {
  preset: 'ts-jest',
  testEnvironment: 'node',
  testMatch: ['**/tests/**/*.test.ts'],
  // Increase timeout for slower parallel rendering
  testTimeout: 60000,
  verbose: true,
  // Cap at 3 parallel browser windows
  maxWorkers: 3, 
};
```

---

## 3. The Golden Rule of Parallel UI Automation

**Thread Safety:**
For parallel execution to work, every test file **must instantiate its own unique WebDriver instance**.

If you share a single `driver` globally across multiple files, the tests will overwrite each other's commands, causing chaos!

Notice how in all of our previous tutorials, we always declared a local driver inside the `describe` block:

```typescript
describe("Core UI Automation", () => {
  let driver: WebDriver; // Local to this file!
  beforeAll(async () => {
    driver = await new Builder().forBrowser("chrome").build();
  });
  // ...
});
```
This architectural decision ensures complete thread-safety.

---

## 4. Test Execution Output

Let's run our entire suite! Since we have 8 test files (`alerts`, `forms`, `iframes`, `mouse_actions`, etc.), Jest will spawn 3 Chrome instances at a time. As soon as one finishes, it will pull the next test from the queue.

```text
> mcyt-sel-typescript@1.0.0 test
> jest
 RUNS  tests/alerts.test.ts
 RUNS  tests/forms.test.ts
 RUNS  tests/wait_strategies.test.ts
 PASS  tests/alerts.test.ts (7.2 s)
 PASS  tests/forms.test.ts (8.1 s)
 PASS  tests/wait_strategies.test.ts (9.5 s)
 RUNS  tests/upload_download.test.ts
 RUNS  tests/windows.test.ts
 RUNS  tests/iframes.test.ts
 PASS  tests/iframes.test.ts (6.3 s)
 PASS  tests/windows.test.ts (7.1 s)
 PASS  tests/upload_download.test.ts (8.4 s)
 RUNS  tests/shadow_dom.test.ts
 RUNS  tests/cdp_network.test.ts
 PASS  tests/cdp_network.test.ts (5.9 s)
 PASS  tests/shadow_dom.test.ts (6.1 s)
Test Suites: 8 passed, 8 total
Tests:       8 passed, 8 total
Snapshots:   0 total
Time:        25.120 s
Ran all test suites.
```

## Conclusion

By simply configuring `maxWorkers: 3` and ensuring WebDriver instances are isolated per file, we reduced our execution time from potentially ~60 seconds (running 8 files sequentially) down to **25 seconds**!

In our final tutorial for this phase, we will learn how to parameterize our framework to perform **Cross-Browser Testing** on Chrome, Firefox, and Edge!
