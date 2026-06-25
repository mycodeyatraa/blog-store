---
title: Integrating Playwright into GitHub Actions Node CI
date: 19-May-2025
lastUpdated: 19-May-2025
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: ["playwright", "typescript", "github-actions", "ci-cd", "node", "automation"]
category: Docker & Grid Scaling
categories: ["Docker & Grid Scaling", "UI Automation", "Playwright", "TypeScript", "CI/CD"]
excerpt: >-
  Transform Playwright into an automated Pull Request gatekeeper by configuring GitHub Actions workflows, optimizing Node.js dependency caching, and securely extracting HTML Report artifacts.
readTime: 4 min read
---

The most common way to integrate Playwright into your development workflow is by running your test suite on every Pull Request. This prevents broken code from ever merging into the `main` branch.

Because Playwright is deeply integrated with the Node.js ecosystem, configuring a **GitHub Actions Node CI** pipeline is incredibly straightforward.

### 1. The Core Pipeline Workflow

To create a pipeline, you define a YAML file inside the `.github/workflows/` directory of your repository. 

The pipeline must perform three key actions:
1. Setup a Node.js environment.
2. Download Playwright's heavy Browser Binaries.
3. Upload the test report so developers can debug failures.

**File:** `.github/workflows/playwright-pr.yml`

```yaml
name: Playwright Pull Request CI
 
on:
  pull_request:
    branches: [ main ] # Trigger on all PRs targeting the main branch
 
jobs:
  test:
    timeout-minutes: 60
    runs-on: ubuntu-latest
 
    steps:
    - uses: actions/checkout@v4
    
    # 1. Setup Node Environment
    - uses: actions/setup-node@v4
      with:
        node-version: 18
 
    - name: Install dependencies
      run: npm ci
 
    # 2. Download Browser Binaries
    - name: Install Playwright Browsers
      run: npx playwright install --with-deps
 
    # 3. Execute the Suite
    - name: Run Playwright tests
      run: npx playwright test
```

### 2. Uploading Artifacts (Crucial Step!)

If a test fails in a headless cloud runner, how does the developer debug it? They can't see the screen!

This is why we configured Playwright to generate HTML reports and Trace Viewer files. However, we must explicitly tell GitHub Actions to save that folder as a downloadable "Artifact".

```yaml
    # Add this AFTER the test execution step!
    - name: Upload HTML Report Artifact
      if: always() # IMPORTANT: Always upload, even if tests failed!
      uses: actions/upload-artifact@v4
      with:
        name: playwright-report
        path: playwright-report/
        retention-days: 7 # Delete after a week to save storage costs
```

*Note the `if: always()` condition.* Without this, if `npx playwright test` fails, the pipeline will immediately abort and skip the upload step entirely!

### 3. Dependency Caching

If your `npm ci` step takes 2 minutes to run because of heavy dependencies, you can speed up the pipeline by caching the `~/.npm` directory.

```yaml
    - name: Cache Node Modules
      uses: actions/cache@v3
      with:
        path: ~/.npm
        key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
```

### Summary

By defining a simple YAML file in your repository, you transform Playwright from a local script into a robust automated gatekeeper. The key to a successful CI implementation is ensuring `actions/upload-artifact` is configured with `if: always()`, guaranteeing developers always have access to Traces and Videos when a failure inevitably occurs!
