---
title: Leaving the Localhost: Introduction to GitHub Actions
date: 17-Jan-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [typescript, selenium, ci-cd, github-actions, automation, devops]
category: Selenium TypeScript
categories: [Selenium TypeScript, CI/CD]
excerpt: >-
  Take your test automation off your local machine and into the cloud by building your first Continuous Integration pipeline using GitHub Actions YAML workflows.
readTime: 4 min read
---

# Leaving the Localhost: Introduction to GitHub Actions

Over the last 10 phases, we have built a remarkably complex and robust enterprise Selenium TypeScript architecture. We've mastered DOM interaction, Page Objects, API interception, Accessibility Audits, and Behavior-Driven Development.

But there is a massive problem: **It only runs on your laptop.**

If test automation only runs when a developer manually types `npm run test` on their local machine, it isn't truly "automation." True test automation acts as a continuous quality gate, executing automatically in the cloud every single time code is pushed.

Welcome to Phase 11: Continuous Integration and Continuous Deployment (CI/CD). Our first stop? **GitHub Actions.**

---

## 1. What is CI/CD?

**Continuous Integration (CI)** is the practice of automatically building and testing code every time a developer commits changes to a repository. 
**Continuous Deployment (CD)** is the practice of automatically deploying that tested code to a production environment.

For QA Engineers, CI is our domain. We want our Selenium test suite to run the moment a developer opens a Pull Request, blocking them from merging their code if our tests fail.

## 2. Why GitHub Actions?

Historically, Jenkins was the undisputed king of CI/CD. However, in modern development, cloud-native CI tools have taken over. GitHub Actions is built directly into the GitHub platform, requiring zero server maintenance, no plugins, and offering generous free tiers for public repositories.

It uses simple YAML configuration files to define "Workflows."

---

## 3. Your First Workflow

To use GitHub Actions, you don't need to install anything. You simply create a highly specific folder structure in the root of your repository: `.github/workflows/`.

Let's create our first workflow file at `.github/workflows/basic-test.yml`:

```yaml
name: Basic Test Workflow
# 1. THE TRIGGER: When should this run?
on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]
# 2. THE JOBS: What servers do we need?
jobs:
  build-and-test:
    runs-on: ubuntu-latest # GitHub provides a free Linux VM!
    # 3. THE STEPS: What commands should the server execute?
    steps:
      # Step 1: Download our code onto the Ubuntu VM
      - name: Checkout Code
        uses: actions/checkout@v3
      # Step 2: Install Node.js
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
      # Step 3: Install our NPM packages (Selenium, Cucumber, etc.)
      - name: Install dependencies
        run: npm ci
      # Step 4: Run the test command!
      - name: Run Tests
        run: echo "Running automation suite..."
```

---

## 4. Understanding the YAML

GitHub Actions workflows are built on three primary pillars:

1. **Events (`on`)**: This defines the trigger. In our example, the workflow will automatically start whenever code is pushed to the `main` branch, or whenever a Pull Request targeting the `main` branch is opened.
2. **Runners (`runs-on`)**: GitHub spins up a brand-new, completely isolated Virtual Machine in the cloud just for your test execution. You can choose `ubuntu-latest`, `windows-latest`, or `macos-latest`. (Note: Linux is much cheaper/faster than Windows!).
3. **Actions (`uses`)**: The open-source community provides pre-built "Actions" so you don't have to reinvent the wheel. `actions/checkout` automatically runs all the necessary `git clone` commands to pull your code into the VM.

## Conclusion

By simply placing a YAML file into the `.github/workflows` directory and pushing it to GitHub, we have officially set up a Continuous Integration pipeline! 

However, running `echo "Running automation suite"` isn't exactly a robust testing strategy. 

In our next tutorial, we will explore **Workflow Triggers** in depth, learning how to schedule tests to run automatically at 3:00 AM every night, or how to trigger them manually with a click of a button!
