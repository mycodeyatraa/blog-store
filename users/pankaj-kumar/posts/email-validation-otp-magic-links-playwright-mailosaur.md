---
title: Enterprise Email Validation: Testing OTPs & Magic Links
date: 02-May-2025
lastUpdated: 02-May-2025
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: ["playwright", "typescript", "email", "mailosaur", "otp", "2fa"]
category: Enterprise Validation
categories: ["Enterprise Validation", "UI Automation", "Playwright", "TypeScript", "Integration"]
excerpt: >-
  Automate multi-factor authentication flows by natively extracting OTPs and Magic Links from virtual inboxes using Playwright and Mailosaur.
readTime: 4 min read
---

A critical piece of end-to-end Enterprise validation is ensuring that external communications actually fire. When a user registers, does the verification email arrive? Does the Magic Login link actually work?

Relying on manual testing for emails is slow, and scraping traditional providers like Gmail is notoriously difficult due to Captchas and 2FA. 

To solve this, we use enterprise email testing APIs like **Mailosaur**. Mailosaur gives you a virtual SMTP server where you can generate infinite temporary email addresses and query their inboxes programmatically via an SDK.

### 1. Installation

Install the official Mailosaur Node.js driver into your Playwright project:

```bash
npm install mailosaur
```

### 2. Extracting OTP Codes

The most common use case is extracting a One-Time Password (OTP) from an email to complete a registration flow.

**File:** `tests/email.spec.ts`

```typescript
import { test, expect } from '@playwright/test';
import MailosaurClient from 'mailosaur';
 
const mailosaur = new MailosaurClient(process.env.MAILOSAUR_API_KEY);
const SERVER_ID = process.env.MAILOSAUR_SERVER_ID;
 
test('Extract OTP code from Registration Email', async ({ page }) => {
  // 1. Generate a unique email address for this exact test run
  // Every email sent to *@SERVER_ID.mailosaur.net is caught by the API
  const testEmailAddress = `user-${Date.now()}@${SERVER_ID}.mailosaur.net`;
 
  // 2. Trigger the UI Action that sends the email
  await page.goto('/register');
  await page.fill('#email', testEmailAddress);
  await page.click('#send-otp-btn');
 
  // 3. Query the Mailosaur API
  // The SDK automatically polls the inbox until the email arrives!
  const email = await mailosaur.messages.get(SERVER_ID, {
    sentTo: testEmailAddress
  }, {
    timeout: 20000 // Timeout after 20 seconds
  });
 
  // 4. Assert Email Metadata
  expect(email.subject).toBe('Your Registration OTP Code');
  expect(email.from[0].email).toBe('no-reply@enterprise-app.com');
 
  // 5. Extract the OTP Code using Regex
  const otpMatch = email.html!.body.match(/Your code is: <b>(\d{6})<\/b>/);
  const otpCode = otpMatch![1];
  
  // 6. Complete the UI flow using the extracted code
  await page.fill('#otp-input', otpCode);
  await page.click('#verify-btn');
  await expect(page.locator('.success-banner')).toBeVisible();
});
```

### 3. Clicking Magic Links

Another common pattern is the "Magic Link" login, where the user receives a URL that authenticates them instantly. The Mailosaur SDK natively parses all HTML links found inside the email body!

```typescript
test('Click Magic Login Link inside an Email', async ({ page }) => {
  const testEmailAddress = `magic-${Date.now()}@${SERVER_ID}.mailosaur.net`;
 
  // Simulate sending magic link from UI...
  // ...
 
  // Await the email
  const email = await mailosaur.messages.get(SERVER_ID, {
    sentTo: testEmailAddress
  });
 
  // Extract all links from the email body
  const links = email.html!.links;
  
  // Find the exact link by its visible text!
  const loginLink = links.find(link => link.text === 'Log in to your account');
  expect(loginLink).toBeDefined();
 
  // Navigate Playwright directly to the extracted URL!
  await page.goto(loginLink!.href);
  await expect(page.locator('#dashboard')).toBeVisible();
});
```

### Summary

By treating emails as just another programmable API, you can easily bridge the gap between your application's UI and its external communications. Using tools like Mailosaur directly inside Playwright `.spec.ts` files allows you to automate complex 2FA and Magic Link workflows natively!
