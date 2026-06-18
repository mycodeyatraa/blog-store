---
title: The Headless Problem: Setting up the Selenium CI Pipeline
date: 19-Jan-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [typescript, selenium, ci-cd, github-actions, headless, xvfb, ubuntu]
category: Selenium TypeScript
categories: [Selenium TypeScript, CI/CD]
excerpt: >-
  Solve the missing GUI problem by configuring headless mode and Xvfb virtual framebuffers to run Selenium TypeScript tests successfully on cloud Ubuntu servers.
readTime: 4 min read
---

# The Headless Problem: Setting up the Selenium CI Pipeline

In our first two tutorials, we learned how to trigger a GitHub Actions workflow that spun up a cloud-hosted Ubuntu Virtual Machine. 

However, if we try to run our Selenium TypeScript code on that basic Ubuntu server, it will instantly crash with a fatal error:

`SessionNotCreatedError: cannot connect to chrome at 127.0.0.1...`

Why? Because Selenium attempts to open the Google Chrome browser window. But an Ubuntu Server in a data center has no monitor, no graphics card, and no graphical user interface (GUI). It is purely a command-line terminal.

You cannot open a browser window if there is no screen.

---

## 1. The Headless Argument

The easiest solution to this problem is to tell Google Chrome not to bother rendering the GUI at all. We do this by adding the `--headless` argument to our Chrome Options in our `hook.ts` file:

```typescript
import { Options } from "selenium-webdriver/chrome";
import { Builder } from "selenium-webdriver";
const chromeOptions = new Options();
chromeOptions.addArguments("--headless=new"); // Do not render the GUI
chromeOptions.addArguments("--no-sandbox"); // Required for Docker/CI environments
chromeOptions.addArguments("--disable-dev-shm-usage"); // Prevents out-of-memory crashes on Linux
driver = await new Builder()
  .forBrowser("chrome")
  .setChromeOptions(chromeOptions)
  .build();
```

With `--headless=new` applied, Chrome will run silently in the background of the Ubuntu VM, executing our tests without attempting to spawn a window!

---

## 2. The Xvfb Alternative (Simulating a Screen)

Sometimes, `--headless` isn't an option. Some highly complex web applications behave slightly differently in headless mode, or perhaps you want to record a video of the test execution, which requires an actual screen.

If you cannot use headless mode, you must use **Xvfb (X Virtual Framebuffer)**.

Xvfb is a Linux program that fakes a monitor. It creates a virtual screen in memory, fooling Google Chrome into thinking there is an actual monitor attached to the server!

We can easily set this up in GitHub Actions using a community-built Action:

```yaml
      # Step: Setup Xvfb so Chrome thinks there is a monitor
      - name: Setup Virtual Display
        uses: GabrielBB/xvfb-action@v1
        with:
          run: npm run test:bdd:smoke
```

Instead of running `npm run` directly, we tell the `xvfb-action` to wrap our test command. It launches the fake display, runs the tests, and then gracefully shuts down!

---

## 3. The Complete CI Pipeline

Now that we understand how to handle the lack of a monitor, let's look at the complete, production-ready `selenium-pipeline.yml` file:

```yaml
name: Selenium E2E Pipeline
on:
  pull_request:
    branches: [ "main" ]
  workflow_dispatch:
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Code
        uses: actions/checkout@v3
      - name: Setup Node.js 18
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm' # Speeds up execution by caching node_modules!
      - name: Install Google Chrome
        # We must explicitly install Chrome on the raw Ubuntu VM
        run: |
          sudo apt-get update
          sudo apt-get install -y google-chrome-stable
      - name: Install NPM Packages
        run: npm ci
      - name: Run Selenium Tests (Headless)
        # Note: Ensure your TypeScript code has `--headless=new` configured!
        run: npm run test:bdd:smoke
```

## Conclusion

You have successfully migrated your test automation from your laptop to the cloud! Your tests will now execute perfectly on a Linux server without crashing due to missing displays.

However, running 500 regression tests sequentially on a single Linux VM is incredibly slow. Your developers will be furious if they have to wait 2 hours for the CI pipeline to finish.

In our next tutorial, we will dramatically speed up our execution times by learning how to configure **Parallel Execution in CI** using GitHub Actions Matrices!
