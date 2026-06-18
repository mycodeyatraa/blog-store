---
title: The Need for Speed: Matrix Parallel Execution in GitHub Actions
date: 20-Jan-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [typescript, selenium, ci-cd, github-actions, parallel, matrix, cross-browser, sharding]
category: Selenium TypeScript
categories: [Selenium TypeScript, CI/CD]
excerpt: >-
  Dramatically reduce UI Automation execution times by utilizing GitHub Actions Matrices to spin up dozens of simultaneous Virtual Machines for cross-browser testing and test sharding.
readTime: 4 min read
---

# The Need for Speed: Matrix Parallel Execution in GitHub Actions

One of the biggest complaints developers have about UI Automation is speed. A comprehensive Selenium Regression suite covering hundreds of User Journeys can easily take 2 to 3 hours to complete. 

If a developer pushes a hotfix, they cannot wait 3 hours for the pipeline to finish before they merge their code.

How do we reduce a 3-hour execution time down to 15 minutes? **Parallel Execution.**

---

## 1. The Strategy: GitHub Actions Matrices

In the past, we learned how to run tests in parallel using the `jest` concurrent runner. However, that approach was limited by the CPU cores on a single machine.

In the cloud, we have infinite machines. 

A **Matrix** in GitHub Actions allows you to define an array of variables. GitHub will then automatically spin up a brand-new, completely independent Ubuntu Virtual Machine for *every single variable* in that array, and run the jobs simultaneously!

---

## 2. Cross-Browser Testing with Matrices

Let's say we want to run our UI Automation suite against Google Chrome, Mozilla Firefox, and Microsoft Edge. 

Instead of writing three different jobs, we use a `matrix`:

```yaml
name: Cross-Browser Parallel Pipeline
on:
  pull_request:
    branches: [ "main" ]
jobs:
  cross-browser-tests:
    runs-on: ubuntu-latest
    # 1. Define the Matrix
    strategy:
      matrix:
        browser: [chrome, firefox, edge]
    steps:
      - uses: actions/checkout@v3
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
      - name: Install dependencies
        run: npm ci
      # 2. Inject the Matrix variable into the test execution command!
      - name: Run Tests
        env:
          BROWSER: ${{ matrix.browser }}
        run: npm run test:bdd
```

### What happens when this runs?
GitHub Actions will read the matrix array `[chrome, firefox, edge]` and immediately spin up **three separate Ubuntu servers**. 
- Server 1 will have the `BROWSER` environment variable set to `chrome`.
- Server 2 will have the `BROWSER` environment variable set to `firefox`.
- Server 3 will have the `BROWSER` environment variable set to `edge`.

They will all execute at the exact same time, effectively turning a 3-hour cross-browser suite into a 1-hour cross-browser suite!

---

## 3. Sharding: Splitting the Suite

Cross-browser parallelization is great, but what if you just want to run Chrome tests as fast as possible? 

We can use a matrix to split our suite into "Shards". For example, if we have 100 tests, we can spin up 4 servers and tell each server to only execute 25 tests!

```yaml
jobs:
  sharded-tests:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        shardIndex: [1, 2, 3, 4] # Spin up 4 servers!
        shardTotal: [4]
    steps:
      - uses: actions/checkout@v3
      - run: npm ci
      - name: Run Tests (Sharded)
        # We pass the shard index into the test runner
        run: npx cucumber-js --order random --parallel ${{ matrix.shardTotal }}
```
*(Note: Sharding implementation depends heavily on the test runner you are using. Jest handles sharding automatically using the `--shard` CLI argument, while Cucumber requires specific configuration).*

## Conclusion

Matrices are the absolute superpower of cloud-native CI pipelines. They allow you to scale your execution horizontally across dozens of virtual machines, turning monolithic, slow automation suites into lightning-fast quality gates.

However, running tests in the cloud introduces a massive security risk. Your tests require passwords to log into the application, and API tokens to seed the database. If you hardcode these credentials into your repository, hackers will find them in seconds.

In our next tutorial, we will learn how to securely manage **Secrets** within GitHub Actions!
