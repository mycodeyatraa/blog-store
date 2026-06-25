---
title: Scaling Playwright with BrowserStack Cloud Grids
date: 21-May-2025
lastUpdated: 21-May-2025
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: ["playwright", "typescript", "browserstack", "cloud", "scaling", "parallel"]
category: Docker & Grid Scaling
categories: ["Docker & Grid Scaling", "UI Automation", "Playwright", "TypeScript", "Infrastructure"]
excerpt: >-
  Unlock massive parallel execution and cross-browser testing by routing Playwright CDP commands through secure WebSockets to the BrowserStack device farm.
readTime: 5 min read
---

Playwright is incredibly fast, but it is limited to modern browser engines (Chromium, WebKit, and Firefox). If your enterprise application requires validation against older browsers (like Internet Explorer 11), specific mobile devices (real iPhones), or different operating systems, you need a Device Farm.

**BrowserStack** is the industry standard cloud grid. Instead of running Playwright browsers locally, we can configure Playwright to connect to a remote BrowserStack server via a WebSocket (`wss://`).

### 1. The CDP Connection Strategy

Playwright communicates with browsers using the **Chrome DevTools Protocol (CDP)**. BrowserStack provides a special WebSocket endpoint that allows Playwright to send these CDP commands securely over the internet.

We create a dedicated configuration file to manage these remote connections.

**File:** `browserstack.config.ts`

```typescript
import { defineConfig } from '@playwright/test';
 
// Helper function to build the BrowserStack CDP Endpoint
const getCdpEndpoint = (browser: string, os: string, osVersion: string) => {
  const caps = {
    'browser': browser,
    'os': os,
    'os_version': osVersion,
    'name': 'Playwright Execution',
    'build': 'build-1.0.0',
    'browserstack.username': process.env.BROWSERSTACK_USERNAME,
    'browserstack.accessKey': process.env.BROWSERSTACK_ACCESS_KEY,
  };
  // Encode the capabilities into the connection URL
  return `wss://cdp.browserstack.com/playwright?caps=${encodeURIComponent(JSON.stringify(caps))}`;
};
 
export default defineConfig({
  testDir: './tests',
  workers: 5, // Execute 5 tests simultaneously on the grid!
  
  projects: [
    {
      name: 'browserstack-chrome-win11',
      use: {
        connectOptions: {
          wsEndpoint: getCdpEndpoint('chrome', 'Windows', '11'),
        },
      },
    },
    {
      name: 'browserstack-safari-mac',
      use: {
        connectOptions: {
          wsEndpoint: getCdpEndpoint('playwright-webkit', 'OS X', 'Ventura'),
        },
      },
    },
  ],
});
```

### 2. Execution and Scale

Notice the `connectOptions.wsEndpoint` property inside the `use` block. When you run this configuration, Playwright will NOT launch a browser on your local laptop. Instead, it securely connects to BrowserStack's data center, spins up a Windows 11 virtual machine, launches Chrome, and executes your test remotely!

To run it, simply pass your credentials into the CLI:

```bash
BROWSERSTACK_USERNAME=my_user BROWSERSTACK_ACCESS_KEY=my_key npx playwright test --config=browserstack.config.ts
```

### 3. Testing Localhost Applications

If you are running a test against `http://localhost:3000` on your laptop, BrowserStack's remote servers cannot see it (because localhost is your machine!). 

To fix this, you must download the **BrowserStackLocal** binary. This establishes a secure VPN tunnel between your laptop and BrowserStack, allowing the remote cloud grid to access your internal staging environments or localhost URLs seamlessly!

### Summary

Connecting Playwright to BrowserStack unlocks limitless scalability. You can bypass the computational limits of your local CI runner and parallelize 100 tests simultaneously across real devices, turning a 30-minute regression suite into a 3-minute execution!
