---
title: Intercepting SMTP: Automated Email Validation
date: 23-Jan-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [typescript, selenium, email-validation, mailtrap, smtp, api, cheerio, enterprise-validation]
category: Selenium TypeScript
categories: [Selenium TypeScript, Enterprise Validation]
excerpt: >-
  Stop automating Gmail logins. Learn how to intercept system-generated emails at the API layer using Mailtrap, parse HTML bodies with Cheerio, and reliably automate the Forgot Password flow.
readTime: 4 min read
---

# Intercepting SMTP: Automated Email Validation

One of the most common—and most difficult—flows to automate in UI testing is the **"Forgot Password"** or **"Account Verification"** loop.

The flow usually looks like this:
1. The user enters their email on the UI.
2. The backend sends an email containing a secure token or link.
3. The user opens the email, clicks the link, and returns to the UI.

How do you automate Step 3? 

Junior engineers will often try to use Selenium to log into Gmail or Outlook. **Never do this.** Major email providers actively block automated logins using CAPTCHAs, 2FA, and IP blacklists. Your tests will be incredibly flaky.

Instead, Enterprise Automation Engineers intercept the emails at the API layer using a dummy SMTP server like **Mailtrap**.

---

## 1. The Mailtrap Concept

Mailtrap (or similar services like Mailhog) provides a fake SMTP server. You configure your staging environment to route *all* outgoing emails to Mailtrap instead of real user inboxes. 

Mailtrap catches the emails and exposes a REST API, allowing your Selenium tests to query the inbox and extract the contents!

---

## 2. Installing Dependencies

We will use `axios` to query the Mailtrap API, and `cheerio` to parse the HTML body of the email to extract the reset link.

```bash
npm install axios cheerio
npm install --save-dev @types/cheerio
```

---

## 3. Creating the Email Interceptor DAO

Let's build a utility class that queries the Mailtrap API, waits for an email to arrive for a specific user, and extracts the magic reset link.

First, set up your `.env`:

```env
MAILTRAP_API_TOKEN=your_secret_api_token
MAILTRAP_INBOX_ID=1234567
```

Create `EmailManager.ts`:

```typescript
import axios from 'axios';
import * as cheerio from 'cheerio';
import * as dotenv from 'dotenv';
dotenv.config();
export class EmailManager {
  private static readonly BASE_URL = `https://mailtrap.io/api/v1/inboxes/${process.env.MAILTRAP_INBOX_ID}`;
  private static readonly HEADERS = {
    'Api-Token': process.env.MAILTRAP_API_TOKEN,
  };
  /**
   * Polls the inbox until an email with a specific subject arrives for the target email address.
   */
  static async waitForEmail(targetEmail: string, subjectLine: string, timeoutMs: number = 15000): Promise<any> {
    const startTime = Date.now();
    while (Date.now() - startTime < timeoutMs) {
      // 1. Fetch all emails in the inbox
      const response = await axios.get(`${this.BASE_URL}/messages`, { headers: this.HEADERS });
      const emails = response.data;
      // 2. Search for our specific email
      const targetMessage = emails.find((msg: any) => 
        msg.to_email === targetEmail && msg.subject === subjectLine
      );
      if (targetMessage) {
        return targetMessage; // Found it!
      }
      // Wait 2 seconds before polling again
      await new Promise(res => setTimeout(res, 2000));
    }
    throw new Error(`Timed out waiting for email to: ${targetEmail}`);
  }
  /**
   * Fetches the HTML body of an email and extracts the reset link using Cheerio.
   */
  static async getResetLinkFromEmail(messageId: string): Promise<string> {
    const response = await axios.get(`${this.BASE_URL}/messages/${messageId}/body.html`, { headers: this.HEADERS });
    // Parse the HTML
    const $ = cheerio.load(response.data);
    // Find the link with the ID or Class used by your developers
    const resetLink = $('a#reset-password-btn').attr('href');
    if (!resetLink) throw new Error('Reset link not found in email HTML body');
    return resetLink;
  }
}
```

---

## 4. The E2E Password Reset Test

Now, we orchestrate the entire flow. We use Selenium to trigger the email, we use our `EmailManager` to intercept the SMTP message via API, we extract the link, and we tell Selenium to navigate to that newly discovered URL!

```typescript
import { Builder, WebDriver } from 'selenium-webdriver';
import { expect } from 'chai';
import { ForgotPasswordPage } from '../pages/ForgotPasswordPage';
import { ResetPasswordPage } from '../pages/ResetPasswordPage';
import { EmailManager } from '../utils/EmailManager';
describe('End-to-End Email Verification', function () {
  let driver: WebDriver;
  const testEmail = `user_${Date.now()}@test.com`;
  before(async function () {
    driver = await new Builder().forBrowser('chrome').build();
  });
  after(async function () {
    await driver.quit();
  });
  it('should complete the forgot password flow via SMTP interception', async function () {
    const forgotPwdPage = new ForgotPasswordPage(driver);
    await forgotPwdPage.navigate();
    // 1. UI Layer Action: Trigger the email
    await forgotPwdPage.requestResetFor(testEmail);
    const confirmation = await forgotPwdPage.getConfirmationMessage();
    expect(confirmation).to.equal('Check your inbox!');
    // 2. API Layer Action: Intercept the Email
    console.log(`Polling Mailtrap for ${testEmail}...`);
    const emailMessage = await EmailManager.waitForEmail(testEmail, 'Password Reset Request');
    expect(emailMessage).to.not.be.null;
    // 3. Extraction: Parse HTML to get the unique token URL
    const resetLink = await EmailManager.getResetLinkFromEmail(emailMessage.id);
    console.log(`Extracted Link: ${resetLink}`);
    // 4. Return to UI Layer: Navigate to the extracted link!
    await driver.get(resetLink);
    const resetPwdPage = new ResetPasswordPage(driver);
    await resetPwdPage.setNewPassword('SecureNewPass123!');
    const success = await resetPwdPage.getSuccessMessage();
    expect(success).to.equal('Password updated successfully');
  });
});
```

## Conclusion

By treating email as an API integration rather than a UI task, we eliminate the flakiness of interacting with third-party webmail providers. Tools like Mailtrap and Cheerio allow us to parse HTML payloads in milliseconds, keeping our CI/CD pipelines lightning fast.

We are almost at the end of our Enterprise Validation phase. Next, we will tackle the manipulation and assertion of raw binary files: **PDF Validation**.
