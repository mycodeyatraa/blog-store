---
title: Configuring CI/CD for Jest/SuperTest
date: 05-Mar-2025
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [supertest, jest, ci-cd, github-actions, devops]
category: API Testing
categories: [API Testing, NodeJS, DevOps, Automation]
excerpt: >-
  Never merge a broken API again. Learn how to configure GitHub Actions to automatically execute your SuperTest suite on every pull request.
readTime: 5 min read
---

Writing a massive SuperTest suite is pointless if developers forget to run it before deploying code. To guarantee that broken APIs never reach production, you must automate the execution of your test suite using a CI/CD pipeline.

Today, we will learn how to configure **GitHub Actions** to automatically run our Jest + SuperTest framework on every commit.

---

## 1. Creating the GitHub Actions Workflow

In the root of your repository, create a directory called `.github/workflows/` and add a new file named `test.yml`. 

This YAML file acts as the blueprint for GitHub's cloud servers, telling them exactly how to spin up a Node.js environment, install our dependencies, and execute our tests.

```yaml
# .github/workflows/test.yml
name: API Automated Tests
# Trigger the workflow on all pushes to the main branch and all pull requests
on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]
jobs:
  test:
    # Run the tests on the latest Ubuntu runner
    runs-on: ubuntu-latest
    steps:
    # 1. Checkout the repository code
    - name: Checkout Code
      uses: actions/checkout@v3
    # 2. Setup Node.js Environment
    - name: Use Node.js 18.x
      uses: actions/setup-node@v3
      with:
        node-version: 18.x
        cache: 'npm' # Automatically cache node_modules to speed up builds
    # 3. Install Dependencies
    - name: Install Dependencies
      run: npm ci # 'npm ci' is strictly for CI environments, bypassing package.json updates
    # 4. Start the Mock Server in the background
    - name: Start Mock Server
      run: npm run start &
    # 5. Wait for the server to boot
    - name: Wait for API to be ready
      run: npx wait-on http://localhost:8080/api/health
    # 6. Execute the Jest/SuperTest Suite
    - name: Run SuperTest Suite
      run: npm run test:coverage
```

## 2. Breaking Down the Workflow

- **`npm ci`**: Unlike `npm install`, `npm ci` strictly reads your `package-lock.json` and guarantees a 100% deterministic install without accidentally upgrading any minor versions that could break your tests.
- **`npm run start &`**: The trailing `&` is a bash command that tells the server to start silently in the background, freeing up the terminal to execute the next steps.
- **`wait-on`**: Since the server might take a few seconds to bind to the port, `wait-on` repeatedly polls the health-check endpoint. Without this, Jest might fire its first HTTP request before the server is awake, leading to `ECONNREFUSED` errors!

## 3. Reviewing the CI Execution

When a developer opens a Pull Request, GitHub Actions will spawn an isolated server and execute the YAML file. If `npm run test` exits with a code of `1` (a test failure), GitHub will physically block the developer from merging the code!

If everything is green, the developer gets a satisfying checkmark:

```bash
Run npm run test:coverage
> mcyt-api-supertest@1.0.0 test:coverage
> jest --coverage
PASS tests/crud.test.ts
PASS tests/faker.test.ts
PASS tests/auth.test.ts
Test Suites: 3 passed, 3 total
Tests:       12 passed, 12 total
Snapshots:   0 total
Time:        15.112 s
```

## 4. Conclusion

By integrating SuperTest into GitHub Actions, you have successfully transformed a manual script into a fully autonomous gatekeeper. Broken APIs can no longer make it into the `main` branch.

In our absolute final tutorial of the series, we will wrap up everything we've learned by exploring **Best Practices & Folder Architecture** for enterprise-grade SuperTest suites!
