---
title: Cloud Agnosticism: CircleCI & GitLab CI
date: 09-Feb-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [typescript, selenium, devops, gitlab-ci, circleci, ci-cd, automation]
category: Selenium TypeScript
categories: [Selenium TypeScript, DevOps]
excerpt: >-
  Don't tie your framework to a single vendor. Learn the core principles of CI/CD architecture and how to seamlessly translate your automation pipelines into GitLab CI and CircleCI.
readTime: 4 min read
---

# Cloud Agnosticism: CircleCI & GitLab CI

In our previous tutorial, we successfully deployed our Dockerized Selenium framework to GitHub Actions. It is an incredible CI/CD platform.

However, as a Senior QA Architect, you must be **Cloud Agnostic**. 

If you join a Fortune 500 company, they might not use GitHub. They might use GitLab, or CircleCI, or Bitbucket Pipelines. You cannot let your automation framework be permanently tied to a single vendor's specific YAML syntax.

Because we containerized our application using Docker, moving our execution to a different cloud provider is incredibly easy.

---

## 1. The Core Philosophy of CI/CD

Regardless of whether you are using GitHub Actions, GitLab CI, or CircleCI, every modern CI/CD pipeline follows the exact same three-step philosophy:

1. **The Executor:** Define the main Virtual Machine (or container) that will run the scripts.
2. **The Services:** Define the background containers (like our Selenium Grid) that the Executor needs to talk to.
3. **The Steps:** Check out the code, install dependencies, run the tests, and save the artifacts.

Let's translate our GitHub Actions workflow into GitLab and CircleCI.

---

## 2. GitLab CI Configuration

GitLab CI uses a file named `.gitlab-ci.yml` in the root of the repository.

Instead of `services` like GitHub, GitLab uses the exact same keyword: `services`!

```yaml
# .gitlab-ci.yml
stages:
  - test
ui-automation:
  stage: test
  # 1. The Executor
  image: node:20-alpine
  # 2. The Services (Selenium Grid)
  services:
    - name: selenium/standalone-chrome:latest
      alias: selenium-grid
  # 3. Environment Variables
  variables:
    GRID_URL: "http://selenium-grid:4444/wd/hub"
    TEST_ENV: "QA"
  # 4. The Steps
  script:
    - echo "Installing dependencies..."
    - npm ci
    - echo "Running Selenium Tests!"
    - npm run test:bdd
  # 5. Capture Artifacts
  artifacts:
    when: always
    paths:
      - allure-results/
```

Notice how much simpler this is! Because we used the `selenium/standalone-chrome` image, GitLab automatically networks the Node.js container to the Selenium container via the `alias`.

---

## 3. CircleCI Configuration

CircleCI uses a `.circleci/config.yml` file. 

CircleCI is famous for its powerful caching mechanisms and complex workflow orchestration.

```yaml
# .circleci/config.yml
version: 2.1
jobs:
  ui-automation:
    # 1. The Executor & Services (CircleCI groups them together!)
    docker:
      - image: cimg/node:20.0
        environment:
          GRID_URL: "http://localhost:4444/wd/hub"
          TEST_ENV: "QA"
      - image: selenium/standalone-chrome:latest
    # 2. The Steps
    steps:
      - checkout
      - run:
          name: Install Dependencies
          command: npm ci
      # 3. Wait for Grid (CircleCI requires dockerize utility)
      - run:
          name: Wait for Selenium Grid
          command: dockerize -wait tcp://localhost:4444 -timeout 1m
      - run:
          name: Execute Tests
          command: npm run test:bdd
      # 4. Capture Artifacts
      - store_artifacts:
          path: allure-results
          destination: allure-results
workflows:
  nightly-regression:
    jobs:
      - ui-automation
```

In CircleCI, the primary container (Node) and the service containers (Selenium) share the same network space, which is why we can use `localhost:4444` for the `GRID_URL`.

## Conclusion

By studying these different YAML configurations, you realize that CI/CD platforms are just different flavors of the exact same architectural concepts. 

Because we decoupled our Node.js executor from the Selenium Grid using Docker, we can easily migrate our framework across any cloud provider on the planet.

But there is a limitation to this architecture. Spinning up a standalone Chrome container is fine for 5 tests. But what if we need to run 5,000 tests across 100 parallel Chrome instances? A single GitHub Runner or GitLab VM will crash.

In our next tutorial, we will solve the ultimate scaling problem: **Dockerized Grid & Kubernetes (K8s) Scaling**.
