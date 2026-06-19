---
title: Automating the Cloud: GitHub Actions for Selenium
date: 08-Feb-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [typescript, selenium, devops, github-actions, ci-cd, docker, automation]
category: Selenium TypeScript
categories: [Selenium TypeScript, DevOps]
excerpt: >-
  Transition your local Docker executions to the cloud. Learn how to write the perfect GitHub Actions YAML workflow to orchestrate a serverless Selenium Grid and execute Pull Request automation.
readTime: 4 min read
---

# Automating the Cloud: GitHub Actions for Selenium

In our previous tutorial, we packaged our entire Selenium TypeScript framework into an immutable Docker container. 

But if we are still manually typing `docker run` on our laptops, we haven't achieved **Continuous Integration**. Continuous Integration means the tests run automatically every time a developer touches the code.

Today, we move our execution to the cloud using **GitHub Actions**.

---

## 1. The GitHub Actions Architecture

GitHub Actions is a CI/CD platform built directly into GitHub. 

When an event occurs (like a developer opening a Pull Request), GitHub spins up a Virtual Machine (called a "Runner"). This Runner is a completely blank slate. You provide a YAML configuration file that tells the Runner exactly what to do: download the code, install Node.js, spin up a Selenium Grid, and execute the tests.

---

## 2. The Complete CI Workflow

Create a file in your repository at `.github/workflows/selenium-ci.yml`.

Here is the ultimate workflow for a Dockerized Selenium TypeScript framework:

```yaml
name: Selenium UI Tests
# 1. When should this workflow run?
on:
  pull_request:
    branches: [ main ]
  schedule:
    - cron: '0 2 * * *' # Run Nightly at 2:00 AM
jobs:
  ui-automation:
    # 2. Spin up an Ubuntu Linux VM
    runs-on: ubuntu-latest
    # 3. Define the Services (The Selenium Grid)
    services:
      selenium-hub:
        image: selenium/hub:latest
        ports:
          - 4444:4444
      chrome-node:
        image: selenium/node-chrome:latest
        env:
          SE_EVENT_BUS_HOST: selenium-hub
          SE_EVENT_BUS_PUBLISH_PORT: 4442
          SE_EVENT_BUS_SUBSCRIBE_PORT: 4443
    steps:
      # 4. Check out the repository code
      - name: Checkout Code
        uses: actions/checkout@v4
      # 5. Wait for the Selenium Grid to be healthy
      - name: Wait for Grid
        run: |
          while ! curl -s http://localhost:4444/wd/hub/status | grep -q '"ready": true'; do
            echo "Waiting for Grid..."
            sleep 2
          done
      # 6. Build the Test Executor Docker Image
      - name: Build Docker Image
        run: docker build -t my-selenium-tests .
      # 7. Execute the Tests!
      - name: Run Tests
        run: |
          docker run --network host \
            -e GRID_URL=http://localhost:4444/wd/hub \
            -e TEST_ENV=QA \
            my-selenium-tests:latest npm run test:bdd
```

---

## 3. Breaking Down the Magic

Let's look at what is actually happening in this YAML file:

1. **Services:** GitHub Actions has a native `services` block. This allows us to spin up background Docker containers before our main steps execute. We use this to spin up a standalone Selenium Hub and a Chrome Node!
2. **Waiting for the Grid:** Selenium containers take a few seconds to boot up. If we run our tests immediately, they will crash with a `ConnectionRefused` error. The `Wait for Grid` step uses a `curl` loop to ping the Grid until it reports `"ready": true`.
3. **The `--network host` Flag:** When we run our test container in step 7, we pass `--network host`. This tells our test container to share the same network as the GitHub Runner VM, allowing it to easily connect to the Selenium Hub at `localhost:4444`.

---

## 4. Capturing Artifacts

If a test fails in the cloud, you cannot see the browser. You need the screenshots and the Allure report.

To extract files out of the ephemeral GitHub Runner before it is destroyed, we use the `upload-artifact` action:

```yaml
      # 8. Upload the Allure Results
      - name: Upload Allure Results
        if: always() # Run even if the tests fail!
        uses: actions/upload-artifact@v4
        with:
          name: allure-results
          path: ./allure-results
```

Now, when a Pull Request fails, the developer simply clicks "Download Artifacts" on the GitHub page to retrieve the failure screenshots!

## Conclusion

By combining our custom `Dockerfile` with the `services` architecture of GitHub Actions, we have built a completely autonomous, serverless automation pipeline. 

But GitHub Actions is not the only CI tool in the enterprise world. What if your company uses GitLab, or CircleCI?

In our next tutorial, we will explore the architectural differences between **CircleCI and GitLab CI**, and how to translate our GitHub pipeline to fit any enterprise ecosystem.
