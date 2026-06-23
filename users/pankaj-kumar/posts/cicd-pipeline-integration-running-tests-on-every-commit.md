---
title: CI/CD Pipeline Integration: Running Tests on Every Commit
date: 26-Jan-2025
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [cicd, api-testing, automation, github-actions, devops]
category: API Testing
categories: [API Testing, Automation, DevOps]
excerpt: >-
  An automation script is useless if it only runs on your local laptop. In this tutorial, we will pull everything together by integrating our tests directly into a GitHub Actions CI/CD Pipeline!
readTime: 5 min read
---

Congratulations! Over the past 12 tutorials, you have mastered REST, tested JWT security, engineered chaos, queried GraphQL, established WebSockets, and caught asynchronous Webhooks. 

You now know how to write world-class API automation scripts. But an automation script is useless if it only runs on your local laptop.

In this tutorial, we will pull everything together by integrating the MyCodeYatra Mock API Server directly into a **GitHub Actions CI/CD Pipeline**. We will configure a workflow that automatically starts the mock server and runs your entire test suite every single time a developer pushes new code!

> **Important Resource:** The entire codebase for this tutorial is hosted on our official GitHub Repository. You can find it here: [https://github.com/MYCodeYatra/myct-api-test-server](https://github.com/MYCodeYatra/myct-api-test-server)

---

## Why Run Tests in CI/CD?

Continuous Integration and Continuous Deployment (CI/CD) is the heart of modern software engineering.

When a developer writes a new feature, they commit their code to a central repository like GitHub. A CI/CD pipeline immediately detects that commit, builds the application, and runs every single automated test. 

If a test fails, the pipeline crashes and blocks the developer from merging their broken code into production. By running your mock server tests in CI/CD, you guarantee that no one can ever deploy a bug to your API!

---

## Setting Up GitHub Actions

To create a pipeline in GitHub, you simply create a YAML configuration file inside a special `.github/workflows` directory in your repository.

Let's write a workflow file that spins up an Ubuntu Linux server, installs Node.js, starts our Mock API Server in the background, and then executes our test commands.

Create a file named `api-tests.yml` and paste the following configuration into it:

```yaml
name: API Test Suite
 
on: [push, pull_request]
 
jobs:
  test:
    runs-on: ubuntu-latest
 
    steps:
      - name: Checkout Code
        uses: actions/checkout@v3
 
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: "20"
 
      - name: Install Dependencies
        run: npm ci
 
      - name: Start Mock Server
        run: npm start &
        env:
          PORT: 8080
 
      - name: Wait for Server to Boot
        run: npx wait-on http://localhost:8080/api/health
 
      - name: Run API Tests
        run: npm test
```

---

## Breaking Down the Workflow

Let's look at exactly what this workflow is doing.

The `on: [push, pull_request]` block tells GitHub to trigger this pipeline every time someone pushes code or opens a Pull Request.

The `npm start &` command is the secret sauce. The ampersand `&` tells the Linux server to start the Mock API Server as a background process! This prevents the pipeline from freezing and allows it to move on to the next step.

Because the server takes a second or two to boot up, we use the `npx wait-on` command to pause the pipeline until the server's health check endpoint returns a `200 OK`. 

Finally, once the mock server is confirmed to be running in the background, we execute `npm test` to fire our automation suite against it!

### Wrapping Up

By embedding the MyCodeYatra Mock API Server into your CI/CD pipelines, you can run thousands of complex backend tests without ever needing to provision expensive, dedicated staging environments. Your tests become faster, cheaper, and infinitely more reliable.

In the **final blog** of this series, we will bring everything we have learned into one massive Capstone Project! Get ready to build a comprehensive, end-to-end testing framework.
