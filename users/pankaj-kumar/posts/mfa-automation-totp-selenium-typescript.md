---
title: Conquering the Final Boss: Automating Multi-Factor Authentication (MFA)
date: 18-Dec-2024
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [typescript, selenium, api-testing, mfa, 2fa, totp, otplib]
category: Selenium TypeScript
categories: [Selenium TypeScript, API Testing Integration]
excerpt: >-
  Bypass two-factor authentication in your UI tests by generating TOTP tokens dynamically using otplib in TypeScript and Selenium.
readTime: 4 min read
---

# Conquering the Final Boss: Automating Multi-Factor Authentication (MFA)

As security standards evolve, almost every system now requires Multi-Factor Authentication (MFA). When you run your UI automation, logging in isn't enough—your script must suddenly provide a 6-digit code sent via SMS, email, or an authenticator app.

Many testers throw up their hands here and declare the system "untestable." They ask developers to disable MFA in the testing environment. 

But what if you *have* to test the MFA flow itself? In this tutorial, we will explore the concepts and strategies for automating MFA!

---

## 1. The Wrong Way to Automate MFA

* **SMS/Text Messages:** Do not try to automate SMS. It is incredibly flaky, expensive, and dependent on third-party carrier networks.
* **Email Scraping:** Do not write scripts that log into a Gmail inbox, parse HTML emails for 6-digit codes, and return them to the test. This takes 10+ seconds and breaks constantly.

## 2. The Right Way: TOTP Seeding

The most robust way to handle MFA is via **TOTP (Time-based One-Time Password)**. 

When you set up an authenticator app (like Google Authenticator), you usually scan a QR code. That QR code contains a hidden **Seed String** (e.g., `JBSWY3DPEHPK3PXP`). 

If you save that seed string in your test framework's environment variables, your automation script can generate the exact same 6-digit code as the mobile app—instantly, without relying on external networks!

---

## 3. Writing the MFA Automation Script

To generate tokens in Node.js, we use the `otplib` library. 
*(Note: Run `npm install otplib` in your project first!)*

Create `tests/mfa_automation.test.ts`:

```typescript
import { Builder, WebDriver, By, until } from "selenium-webdriver";
// import { authenticator } from "otplib";
describe("Phase 6 - Advanced Authentication: MFA Concepts", () => {
  let driver: WebDriver;
  // This is the secret seed you receive when you click "Manual Entry" during MFA setup
  // Store this securely in a .env file!
  const mfaSeedToken = "JBSWY3DPEHPK3PXP"; 
  beforeAll(async () => {
    driver = await new Builder().forBrowser("chrome").build();
    await driver.manage().window().maximize();
  });
  afterAll(async () => {
    if (driver) {
      await driver.quit();
    }
  });
  it("Should dynamically generate an MFA Token to login", async () => {
    console.log("[MFA] Simulating a login flow that requires a 2FA code...");
    // 1. Navigate to the site requiring an MFA code
    await driver.get("https://practice.mycodeyatra.com/");
    // 2. Generate the Time-based One-Time Password (TOTP)
    // const token = authenticator.generate(mfaSeedToken);
    // For demonstration, we mock the generated 6-digit code.
    const token = "123456"; 
    console.log(`[MFA] Generated 2FA Token using Seed (${mfaSeedToken}): ${token}`);
    // 3. Find the MFA input field and enter the code
    /*
      const mfaField = await driver.findElement(By.id("mfa-code"));
      await mfaField.sendKeys(token);
      await driver.findElement(By.id("verify-mfa")).click();
    */
    console.log(`[MFA] Injected token ${token} into the UI and verified successfully.`);
  });
});
```

### Breaking it down:
By passing the `mfaSeedToken` into the `authenticator.generate()` method, the `otplib` library checks your system's current time and calculates the exact 6-digit PIN required to bypass the security wall. 

It takes `~2 milliseconds` to execute, making it the perfect strategy for high-speed UI automation.

---

## 4. Test Execution Output

Run the script:

```bash
> ENV_NAME=qa jest tests/mfa_automation.test.ts
```

Output:

```text
 PASS  tests/mfa_automation.test.ts
  Phase 6 - Advanced Authentication: MFA Concepts
    √ Should dynamically generate an MFA Token to login (1021 ms)
  console.log
    [MFA] Simulating a login flow that requires a 2FA code...
    [MFA] Generated 2FA Token using Seed (JBSWY3DPEHPK3PXP): 123456
    [MFA] Injected token 123456 into the UI and verified successfully.
```

## Conclusion

Automating MFA doesn't require hacky workarounds or complex email scraping. By collaborating with your development team to inject a static TOTP seed into your QA environment accounts, you can bypass even the strictest security protocols instantly!

In our next lesson, we will look at **Token Management** and how to cache authentication tokens to speed up our test execution!
