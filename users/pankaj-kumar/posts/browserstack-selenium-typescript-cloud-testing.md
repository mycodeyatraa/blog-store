---
title: Outsourcing the Grid: BrowserStack and TypeScript
date: 11-Feb-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [typescript, selenium, devops, browserstack, cloud, cross-browser, testing]
category: Selenium TypeScript
categories: [Selenium TypeScript, DevOps]
excerpt: >-
  Stop managing servers and start testing on real devices. Learn how to configure your TypeScript framework to execute tests on the BrowserStack Cloud Grid and securely test internal applications via VPN.
readTime: 4 min read
---

# Outsourcing the Grid: BrowserStack and TypeScript

In our previous tutorial, we explored how to build a massively scalable Selenium Grid using Kubernetes.

While Kubernetes is incredibly powerful, it comes with a massive hidden cost: **Maintenance**. 
When Chrome updates to v125, you have to update your Docker images. When the Linux server runs out of disk space, you have to clear the logs. 

Furthermore, a Kubernetes Grid can only run Docker containers. Docker containers cannot run iOS Safari. They cannot run native Windows Edge.

If your company needs to test on thousands of real physical devices and every browser combination known to man, without hiring a 5-person DevOps team, you must turn to a Cloud Testing Provider. 

The industry leader in this space is **BrowserStack**.

---

## 1. What is BrowserStack?

BrowserStack is essentially a massive, globally distributed Selenium Grid hosted in the cloud. They maintain tens of thousands of real iPhones, Androids, MacBooks, and Windows machines in their data centers.

Instead of pointing your automation framework to `http://localhost:4444`, you point it to `http://hub-cloud.browserstack.com/wd/hub`. 

BrowserStack receives your Selenium commands, executes them on a physical device in their data center, and sends the result back to your local machine!

---

## 2. Integrating with BrowserStack

To use BrowserStack, you need an Account, a Username, and an Access Key.

In your automation framework, you simply update the `Builder` initialization inside your hooks. We use the `bstack:options` capability dictionary to tell BrowserStack exactly which device and OS we want to use.

```typescript
import { Builder } from 'selenium-webdriver';
const USERNAME = process.env.BROWSERSTACK_USERNAME;
const ACCESS_KEY = process.env.BROWSERSTACK_ACCESS_KEY;
const GRID_URL = `https://${USERNAME}:${ACCESS_KEY}@hub-cloud.browserstack.com/wd/hub`;
const capabilities = {
  'bstack:options' : {
    "os" : "Windows",
    "osVersion" : "11",
    "browserVersion" : "latest",
    "projectName" : "E-Commerce-Regression",
    "buildName" : "Nightly-Build-105",
    "sessionName" : "Login Scenarios"
  },
  "browserName" : "Edge"
};
const driver = await new Builder()
  .usingServer(GRID_URL)
  .withCapabilities(capabilities)
  .build();
```

When you run this code, your test will magically execute on a pristine Windows 11 machine running the latest version of Microsoft Edge, located somewhere in a data center in California!

---

## 3. The BrowserStack Local Binary

What if your application is not public? What if you are testing an internal QA environment hosted on `http://localhost:3000`?

BrowserStack servers cannot access your company's internal intranet.

To solve this, BrowserStack provides a tool called **BrowserStack Local**. It establishes a secure VPN tunnel between your local machine (or your GitHub Actions Runner) and the BrowserStack data center.

You start the binary before your tests run:

```bash
./BrowserStackLocal --key $BROWSERSTACK_ACCESS_KEY
```

Then, you add one line to your capabilities:

```typescript
  'bstack:options' : {
    "local" : "true",
    // ...
  }
```
Now, the iPhone sitting in the BrowserStack data center will securely route all its HTTP traffic through your local laptop, allowing it to test your private, unreleased code!

---

## 4. Automatic Video Recording and Logs

The greatest benefit of Cloud Providers is their built-in Observability. 

When a test finishes on BrowserStack, their dashboard automatically provides:
- A full 1080p MP4 Video Recording of the test execution.
- A complete Network Log (every HTTP request the browser made).
- A complete Console Log (every JavaScript error thrown in the browser).

You don't have to configure Allure or the ELK stack to get this data; it comes entirely out-of-the-box!

## Conclusion

BrowserStack allows you to instantly scale your testing to thousands of OS/Browser combinations with zero infrastructure maintenance. 

But BrowserStack is not the only player in the game. In our next tutorial, we will explore one of their biggest competitors: **Sauce Labs**, and learn how to configure our framework to toggle between different cloud providers seamlessly!
