---
title: Scaling Playwright with Sauce Labs Automation Cloud
date: 22-May-2025
lastUpdated: 22-May-2025
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: ["playwright", "typescript", "saucelabs", "cloud", "scaling", "parallel"]
category: Docker & Grid Scaling
categories: ["Docker & Grid Scaling", "UI Automation", "Playwright", "TypeScript", "Infrastructure"]
excerpt: >-
  Execute cross-browser automation at an enterprise scale by natively connecting Playwright's CDP engine directly to the Sauce Labs infrastructure.
readTime: 4 min read
---

While BrowserStack is incredibly popular, **Sauce Labs** is another industry titan in the Enterprise device farm space. If your enterprise contract is with Sauce Labs, porting your Playwright suite to run on their cloud infrastructure follows a very similar WebSocket architecture.

Sauce Labs allows you to execute your Playwright tests using two distinct methods: **SauceCTL** (their proprietary CLI tool) or **CDP WebSockets** (the native Playwright approach). We highly recommend the native approach because it requires zero new dependencies!

### 1. The Sauce Labs WebSocket Architecture

Just like we saw with BrowserStack, we can hijack Playwright's `connectOptions` to point our browser instances at the Sauce Labs datacenter instead of our local machine.

**File:** `sauce.config.ts`

```typescript
import { defineConfig } from '@playwright/test';
 
// Helper function to build the Sauce Labs CDP Endpoint
const getSauceEndpoint = (browserName: string, browserVersion: string, platform: string) => {
  const capabilities = {
    browserName,
    browserVersion,
    platformName: platform,
    'sauce:options': {
      name: 'Playwright Production Sanity',
      build: `build-${Date.now()}`,
      username: process.env.SAUCE_USERNAME,
      accessKey: process.env.SAUCE_ACCESS_KEY,
    }
  };
 
  // Determine datacenter (US West vs EU Central)
  const dataCenter = process.env.SAUCE_REGION === 'eu' ? 'ondemand.eu-central-1.saucelabs.com' : 'ondemand.us-west-1.saucelabs.com';
  
  return `wss://${process.env.SAUCE_USERNAME}:${process.env.SAUCE_ACCESS_KEY}@${dataCenter}/playwright?caps=${encodeURIComponent(JSON.stringify(capabilities))}`;
};
 
export default defineConfig({
  testDir: './tests',
  workers: 10, // Maximize this based on your Enterprise Sauce Labs concurrency limits!
  
  projects: [
    {
      name: 'sauce-chrome-win10',
      use: {
        connectOptions: {
          wsEndpoint: getSauceEndpoint('chrome', 'latest', 'Windows 10'),
        },
      },
    },
  ],
});
```

### 2. Executing the Grid

Once your `sauce.config.ts` is defined, executing it via your terminal or CI/CD pipeline is seamless. 

Ensure your environment variables are exported, and run the standard Playwright CLI:

```bash
SAUCE_USERNAME=your_user SAUCE_ACCESS_KEY=your_key npx playwright test --config=sauce.config.ts
```

### 3. Native Playwright Traces in Sauce Labs

Because we are using pure native Playwright CDP commands, Sauce Labs fully supports Playwright's proprietary debugging artifacts! 

If your Playwright configuration is set to generate a Trace Viewer (`trace: 'on-first-retry'`) or a Video (`video: 'retain-on-failure'`), Sauce Labs will securely capture these files during the remote execution and display them directly inside the Sauce Labs Dashboard under the "Test Results" tab.

### Summary

Integrating Playwright with Sauce Labs via pure CDP WebSockets is the cleanest way to scale your test infrastructure. You completely bypass the limitations of your CI/CD runner's CPU by offloading the actual browser rendering to the Sauce Labs cloud, allowing you to run dozens of cross-browser tests simultaneously!
