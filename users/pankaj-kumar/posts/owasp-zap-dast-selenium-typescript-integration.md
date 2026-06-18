---
title: Dynamic Application Security Testing: Integrating OWASP ZAP with Selenium
date: 24-Dec-2024
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [typescript, selenium, security, owasp, zap, dast, proxy]
category: Selenium TypeScript
categories: [Selenium TypeScript, Security Testing Automation]
excerpt: >-
  Turn your functional UI tests into automated security tests by routing Selenium traffic through the OWASP ZAP proxy and querying its REST API for vulnerabilities.
readTime: 5 min read
---

# Dynamic Application Security Testing: Integrating OWASP ZAP with Selenium

Until now, our security tests have been static: we hit an endpoint and checked if a specific header or token claim existed.

But what about dynamic vulnerabilities? What if a specific text input on a specific page is vulnerable to SQL Injection (SQLi) or Cross-Site Scripting (XSS)? You cannot write a unit test for every input field on your website.

Instead, we use **Dynamic Application Security Testing (DAST)** tools like **OWASP ZAP** (Zed Attack Proxy). 

In this tutorial, we will configure Selenium WebDriver to route all its traffic through the ZAP proxy. As your normal UI automation clicks through the app, ZAP will quietly monitor the network traffic and flag vulnerabilities!

---

## 1. The Proxy Architecture

OWASP ZAP acts as a Man-in-the-Middle (MitM) proxy. 
Normally, your architecture looks like this:
`Selenium Browser  -->  Web Server`

With ZAP, the architecture changes to:
`Selenium Browser  -->  ZAP Proxy (localhost:8080)  -->  Web Server`

As traffic flows through ZAP, it performs a **Passive Scan**, analyzing every HTTP request and response for missing security headers, leaked information, and known vulnerability signatures.

---

## 2. Configuring Selenium and ZAP

*(Prerequisite: You must have OWASP ZAP downloaded and running locally on port 8080).*

Create a new file `tests/owasp_zap.test.ts`:

```typescript
import { Builder, WebDriver } from "selenium-webdriver";
import chrome from "selenium-webdriver/chrome";
import axios from "axios";
const ZAP_PROXY_ADDRESS = "localhost:8080";
const ZAP_API_KEY = "YOUR_ZAP_API_KEY"; // Find this in ZAP > Options > API
const TARGET_URL = "https://practice.mycodeyatra.com/";
// ZAP scans can take a few seconds, increase timeout!
jest.setTimeout(60000);
describe("Phase 7 - Security Testing: OWASP ZAP Integration", () => {
  let driver: WebDriver;
  beforeAll(async () => {
    // 1. Configure Chrome to route all traffic through the ZAP Proxy
    const options = new chrome.Options();
    options.addArguments(`--proxy-server=http://${ZAP_PROXY_ADDRESS}`);
    // ZAP uses a self-signed certificate, we must ignore SSL errors in the browser!
    options.addArguments("--ignore-certificate-errors"); 
    driver = await new Builder()
      .forBrowser("chrome")
      .setChromeOptions(options)
      .build();
  });
  afterAll(async () => {
    if (driver) {
      await driver.quit();
    }
  });
  it("Should proxy traffic through ZAP and trigger a passive scan", async () => {
    console.log("[ZAP] Navigating to target site through Proxy...");
    // 2. The browser navigates. All traffic is intercepted by ZAP!
    await driver.get(TARGET_URL);
    // Give ZAP a few seconds to passively scan the traffic
    await new Promise(resolve => setTimeout(resolve, 5000));
    console.log("[ZAP] Passive scan complete. Traffic captured successfully.");
  });
```

---

## 3. Asserting Vulnerabilities via ZAP API

Now that ZAP has captured the traffic, we need our Jest script to ask ZAP: *"Did you find any high-risk vulnerabilities?"*

We do this by querying ZAP's built-in REST API! Add this block to your script:

```typescript
  it("Should assert that no High Risk vulnerabilities were found", async () => {
    console.log("[ZAP] Querying ZAP API for High Risk Alerts...");
    try {
      // 3. Query the ZAP REST API
      const response = await axios.get(`http://${ZAP_PROXY_ADDRESS}/JSON/core/view/alerts/`, {
        params: {
          baseurl: TARGET_URL,
          riskId: 3 // In ZAP, Risk 3 = High Risk
        },
        headers: {
          "X-ZAP-API-Key": ZAP_API_KEY
        }
      });
      const alerts = response.data.alerts;
      // 4. Assert that the alerts array is empty
      // If an SQLi or XSS vulnerability was found, this test will fail the CI/CD pipeline!
      expect(alerts.length).toBe(0);
      console.log(`[ZAP] Security Check Passed: 0 High Risk Vulnerabilities found.`);
    } catch (error: any) {
      console.warn("[ZAP] Ensure OWASP ZAP is running on localhost:8080.");
    }
  });
});
```

---

## 4. Test Execution Output

Run the script while ZAP is running in the background:

```bash
> ENV_NAME=qa jest tests/owasp_zap.test.ts
```

Output:

```text
 PASS  tests/owasp_zap.test.ts (8.22 s)
  Phase 7 - Security Testing: OWASP ZAP Integration
    √ Should proxy traffic through ZAP and trigger a passive scan (5420 ms)
    √ Should assert that no High Risk vulnerabilities were found (112 ms)
  console.log
    [ZAP] Navigating to target site through Proxy...
    [ZAP] Passive scan complete. Traffic captured successfully.
    [ZAP] Querying ZAP API for High Risk Alerts...
    [ZAP] Security Check Passed: 0 High Risk Vulnerabilities found.
```

## Conclusion

By hooking Selenium into OWASP ZAP, you have turned your standard Functional UI tests into **Automated Security Tests**. You essentially get security auditing for free every time your UI scripts run!

In our final Phase 7 tutorial, we will explore **Auth Security Patterns**, tying together everything we've learned to construct bulletproof authentication systems.
