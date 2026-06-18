---
title: Securing the Pipeline: Handling Secrets in GitHub Actions
date: 21-Jan-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [typescript, selenium, ci-cd, github-actions, secrets, security, env]
category: Selenium TypeScript
categories: [Selenium TypeScript, CI/CD]
excerpt: >-
  Protect your automation infrastructure by removing hardcoded passwords from your TypeScript code and securely injecting GitHub Secrets via Environment Variables.
readTime: 4 min read
---

# Securing the Pipeline: Handling Secrets in GitHub Actions

In our `login.steps.ts` file, we wrote code that looked something like this:

```typescript
await driver.findElement(By.id("password")).sendKeys("MySuperSecretPassword123!");
```

If you commit this code to a public GitHub repository, a bot will scrape that password within 3 seconds of the push. If that password is used for a production database or a third-party API, your company is now compromised.

**Never hardcode passwords in your automation scripts.**

Instead, we must securely pass credentials into our tests at runtime using Environment Variables and GitHub Actions Secrets.

---

## 1. Using Environment Variables in TypeScript

First, we must refactor our TypeScript code. Instead of hardcoding the string, we tell Node.js to read the value from the operating system's environment variables:

```typescript
// Refactored login.steps.ts
When('the user enters the admin password', async function () {
  // Read the password from the environment
  const adminPassword = process.env.ADMIN_PASSWORD;
  if (!adminPassword) {
    throw new Error("CRITICAL: ADMIN_PASSWORD environment variable is missing!");
  }
  await driver.findElement(By.id("password")).sendKeys(adminPassword);
});
```

Now, if a developer runs this code locally, it will instantly crash unless they have explicitly set the `ADMIN_PASSWORD` variable on their laptop (usually via a `.env` file).

But how do we pass this variable to the Ubuntu server in GitHub Actions?

---

## 2. Storing Secrets in GitHub

GitHub provides a secure, encrypted vault for every repository.

1. Navigate to your GitHub Repository in the browser.
2. Click on **Settings**.
3. In the left sidebar, expand **Secrets and variables**, then click **Actions**.
4. Click the green **New repository secret** button.
5. Set the Name to `ADMIN_PASSWORD` and the Value to your actual password.

Once saved, GitHub encrypts the value. You can *never* view it again (you can only update or delete it). Even if you try to `console.log()` the secret inside your CI pipeline, GitHub will automatically mask it in the logs with `***`.

---

## 3. Injecting Secrets into the Workflow

Now that the secret is securely stored in GitHub's vault, we need to inject it into the Ubuntu Virtual Machine right before our tests execute.

We do this using the `env:` block in our YAML file:

```yaml
name: Secure Automation Pipeline
on:
  push:
    branches: [ "main" ]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - run: npm ci
      - name: Run Secure Tests
        # Inject the GitHub Secret into the Node.js Environment!
        env:
          ADMIN_PASSWORD: ${{ secrets.ADMIN_PASSWORD }}
          API_TOKEN: ${{ secrets.PROD_API_TOKEN }}
        run: npm run test:bdd
```

### The Security Flow:
1. GitHub spins up the Ubuntu VM.
2. It reaches into its encrypted vault and extracts the `ADMIN_PASSWORD`.
3. It temporarily assigns that value to the Linux server's environment variables.
4. `npm run test:bdd` executes.
5. Our TypeScript code reads `process.env.ADMIN_PASSWORD` and successfully types it into the browser.
6. The test finishes, the Ubuntu VM is destroyed, and the password is erased from memory!

## Conclusion

Handling secrets properly is the hallmark of a mature DevOps engineer. By utilizing `process.env` and GitHub Secrets, your code remains completely open-source and reviewable, while your production environments remain entirely secure.

However, once our tests finish running, the Virtual Machine is instantly destroyed—and so are our Test Reports!

In our next tutorial, we will learn how to extract HTML test reports and Screenshots out of the Virtual Machine before it explodes, using **GitHub Actions Artifacts**!
