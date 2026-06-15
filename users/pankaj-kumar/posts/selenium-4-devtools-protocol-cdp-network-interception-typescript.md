---
title: Selenium 4 DevTools Network: Intercepting API Traffic in TypeScript
date: 16-Nov-2024
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [typescript, selenium, cdp, devtools, network-interception]
category: Selenium TypeScript
categories: [Selenium TypeScript, UI Automation, Core Automation]
excerpt: >-
  With Selenium 4, Automation Engineers gained native access to the Chrome DevTools Protocol (CDP). Learn how to peer under the hood and intercept API requests.
readTime: 6 min read
---

# Selenium 4 DevTools Network: Intercepting API Traffic in TypeScript

For years, Selenium was restricted solely to the "User Interface." If a backend API failed or returned a 500 Error, Selenium would blindly wait for a UI element to appear, eventually crashing with a misleading `TimeoutError`.

With the release of **Selenium 4**, Automation Engineers gained native access to the **Chrome DevTools Protocol (CDP)**. This revolutionary feature allows us to peer under the hood, intercept Network Requests, block images, and even mock API responses on the fly!

In this tutorial, we will learn how to connect to CDP using TypeScript and capture network traffic from **[practice.mycodeyatra.com](https://practice.mycodeyatra.com/)**.

---

## 1. Connecting to the Chrome DevTools Protocol

To intercept network traffic, we must first establish a WebSocket connection directly to the browser's DevTools backend using `driver.createCDPConnection()`.

```typescript
// Establish CDP Connection
const cdpConnection = await driver.createCDPConnection("page");
// Enable the Network domain
await cdpConnection.execute("Network.enable", {}, (res) => {});
```

---

## 2. Subscribing to Network Events

Once the Network domain is enabled, the browser will begin broadcasting WebSocket messages for every single HTTP request and response. 

We can listen to these raw messages by attaching an event listener to the internal `_wsConnection`.

```typescript
const requests: string[] = [];
// Listen for incoming WebSocket messages
cdpConnection._wsConnection.on("message", (message: string) => {
  const parsedMessage = JSON.parse(message);
  // Intercept Outgoing Requests
  if (parsedMessage.method === "Network.requestWillBeSent") {
    requests.push(parsedMessage.params.request.url);
  }
});
```

---

## 3. Writing the CDP Interception Test

Let's write a complete Jest test. We will enable CDP, set up our listeners, navigate to the practice site, and assert that we successfully captured the backend traffic!

Create `tests/cdp_network.test.ts`:

```typescript
import { Builder, WebDriver } from "selenium-webdriver";
import "chromedriver";
describe("Core UI Automation - Selenium 4 DevTools Network", () => {
  let driver: WebDriver;
  beforeAll(async () => {
    driver = await new Builder().forBrowser("chrome").build();
    await driver.manage().window().maximize();
  });
  afterAll(async () => {
    if (driver) {
      await driver.quit();
    }
  });
  it("Should connect to CDP and intercept Network Requests", async () => {
    // 1. Get the CDP Connection
    const cdpConnection = await driver.createCDPConnection("page");
    // Arrays to store captured network traffic
    const requests: string[] = [];
    const responses: string[] = [];
    // 2. Subscribe to Network Events
    await cdpConnection.execute("Network.enable", {}, (res) => {});
    cdpConnection._wsConnection.on("message", (message: string) => {
      const parsedMessage = JSON.parse(message);
      if (parsedMessage.method === "Network.requestWillBeSent") {
        requests.push(parsedMessage.params.request.url);
      }
      if (parsedMessage.method === "Network.responseReceived") {
        responses.push(parsedMessage.params.response.url);
      }
    });
    console.log("CDP Network Interception Enabled...");
    // 3. Navigate to the Practice Site
    await driver.get("https://practice.mycodeyatra.com/");
    // Wait briefly for network calls to finish
    await driver.sleep(3000);
    console.log(`Captured ${requests.length} total outgoing requests.`);
    console.log(`Captured ${responses.length} total incoming responses.`);
    // Assert we actually captured network traffic
    expect(requests.length).toBeGreaterThan(0);
    expect(responses.length).toBeGreaterThan(0);
    // Verify that the main document was requested
    const mainRequest = requests.find(url => url.includes("practice.mycodeyatra.com"));
    expect(mainRequest).toBeDefined();
    console.log(`Successfully verified interception of: ${mainRequest}`);
  });
});
```

---

## 4. Test Execution Output

```text
> mcyt-sel-typescript@1.0.0 test
> jest
 PASS  tests/cdp_network.test.ts (6.113 s)
  Core UI Automation - Selenium 4 DevTools Network
    √ Should connect to CDP and intercept Network Requests (4011 ms)
  console.log
    CDP Network Interception Enabled...
  console.log
    Captured 12 total outgoing requests.
  console.log
    Captured 12 total incoming responses.
  console.log
    Successfully verified interception of: https://practice.mycodeyatra.com/
Test Suites: 1 passed, 1 total
Tests:       1 passed, 1 total
Snapshots:   0 total
Time:        6.313 s
Ran all test suites.
```

## Conclusion

CDP completely bridges the gap between Frontend UI automation and Backend API validation. Instead of guessing why a page is blank, your scripts can now explicitly check if the `/users/profile` API returned a `500 Internal Server Error`.

In our next tutorial, we will focus on drastically reducing our test execution time by configuring **Jest Parallel Execution!**
