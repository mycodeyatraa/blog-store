---
title: Vendor Agnosticism: Sauce Labs and TypeScript
date: 12-Feb-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [typescript, selenium, devops, saucelabs, cloud, cross-browser, testing]
category: Selenium TypeScript
categories: [Selenium TypeScript, DevOps]
excerpt: >-
  Don't get locked into a single cloud vendor. Learn how to execute your TypeScript framework on the Sauce Labs Grid, use Sauce Connect Proxy, and build a Factory Pattern to seamlessly switch providers.
readTime: 4 min read
---

# Vendor Agnosticism: Sauce Labs and TypeScript

In our previous tutorial, we configured our TypeScript framework to execute tests on the BrowserStack Cloud Grid. It solved our infrastructure problems instantly.

But what happens when your company's contract with BrowserStack expires, and the Procurement Department decides to sign a cheaper contract with **Sauce Labs** instead?

If your framework's architecture is permanently hardcoded to `bstack:options`, you will have to rewrite hundreds of lines of code.

As a Senior QA Architect, you must practice **Vendor Agnosticism**. Your framework should be able to switch Cloud Providers by changing a single Environment Variable.

---

## 1. What is Sauce Labs?

Sauce Labs is the original pioneer of Cloud Testing. They provide the exact same core service as BrowserStack: a massive, globally distributed Selenium Grid hosting thousands of real physical devices.

Just like BrowserStack, you need a Username, an Access Key, and a specific Remote Grid URL.

The Sauce Labs Grid URL looks like this:
`https://ondemand.us-west-1.saucelabs.com:443/wd/hub`

---

## 2. Integrating with Sauce Labs

Sauce Labs uses the exact same W3C WebDriver specification as BrowserStack, but they use a different vendor prefix in their capabilities object: `sauce:options`.

Here is the TypeScript configuration for Sauce Labs:

```typescript
import { Builder } from 'selenium-webdriver';
const USERNAME = process.env.SAUCE_USERNAME;
const ACCESS_KEY = process.env.SAUCE_ACCESS_KEY;
const GRID_URL = `https://${USERNAME}:${ACCESS_KEY}@ondemand.us-west-1.saucelabs.com:443/wd/hub`;
const capabilities = {
  'sauce:options' : {
    "os" : "macOS 13",
    "build" : "Nightly-Build-105",
    "name" : "Login Scenarios"
  },
  "browserName" : "safari",
  "browserVersion" : "latest"
};
const driver = await new Builder()
  .usingServer(GRID_URL)
  .withCapabilities(capabilities)
  .build();
```

Notice the subtle differences. BrowserStack uses `osVersion: "11"` and `os: "Windows"`. Sauce Labs combines them into a single string: `os: "macOS 13"`. BrowserStack uses `sessionName`. Sauce Labs uses `name`.

---

## 3. The Sauce Connect Proxy

If you are testing an internal QA environment that is not accessible from the public internet, Sauce Labs provides a VPN tunneling tool exactly like BrowserStack Local. 

It is called **Sauce Connect Proxy**.

You download the binary and start it in your terminal:

```bash
./sc -u $SAUCE_USERNAME -k $SAUCE_ACCESS_KEY
```

Then, you add one line to your capabilities:

```typescript
  'sauce:options' : {
    "tunnelName" : "my-tunnel",
    // ...
  }
```

---

## 4. Architectural Flexibility

To achieve true Vendor Agnosticism, you should create a `DriverFactory.ts` class that dynamically checks an environment variable:

```typescript
// DriverFactory.ts
export class DriverFactory {
  static async getCloudDriver() {
    const cloudProvider = process.env.CLOUD_PROVIDER; // 'BROWSERSTACK' or 'SAUCELABS'
    if (cloudProvider === 'BROWSERSTACK') {
      return this.buildBrowserStackDriver();
    } else if (cloudProvider === 'SAUCELABS') {
      return this.buildSauceLabsDriver();
    } else {
      throw new Error(`Unsupported Cloud Provider: ${cloudProvider}`);
    }
  }
}
```

Now, your engineers can toggle between Cloud Providers without changing a single line of test code!

## Conclusion

By studying both BrowserStack and Sauce Labs, you understand that all Cloud Providers operate on the exact same underlying W3C WebDriver architecture. They just use different vendor prefixes in their capability objects.

But there is one more major player in the Cloud Testing space that we must analyze before we finish our DevOps phase: **LambdaTest**. We will cover that in our next tutorial!
