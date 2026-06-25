---
title: Executing Playwright in CircleCI and GitLab CI
date: 20-May-2025
lastUpdated: 20-May-2025
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: ["playwright", "typescript", "circleci", "gitlab", "ci-cd", "devops"]
category: Docker & Grid Scaling
categories: ["Docker & Grid Scaling", "UI Automation", "Playwright", "TypeScript", "CI/CD"]
excerpt: >-
  Expand your DevOps toolkit by porting your Playwright automation suite to enterprise CI platforms like CircleCI and GitLab CI using the official Microsoft Docker executor.
readTime: 4 min read
---

While GitHub Actions is incredibly popular, many enterprise organizations rely on **CircleCI** or **GitLab CI** for their DevOps pipelines.

Because Playwright is fundamentally just a Node.js process, porting it to any CI system is straightforward. However, handling caching and artifact uploads requires slightly different syntax depending on the platform.

### 1. Playwright on GitLab CI

GitLab uses a `.gitlab-ci.yml` file placed at the root of the repository. The trick with GitLab is ensuring you use the official Playwright Docker image as the base executor, eliminating the need to install browser binaries manually!

**File:** `.gitlab-ci.yml`

```yaml
stages:
  - test
 
playwright_tests:
  stage: test
  # Use the official image to guarantee OS dependencies are present!
  image: mcr.microsoft.com/playwright:v1.43.0-jammy
  
  script:
    - npm ci
    # Browsers are already in the image, but we install deps just in case
    - npx playwright install --with-deps
    - npx playwright test
    
  artifacts:
    when: always # Upload artifacts even if the tests fail
    paths:
      - playwright-report/
    expire_in: 7 days
```

### 2. Playwright on CircleCI

CircleCI uses a `.circleci/config.yml` file. Like GitLab, we can specify a custom Docker image for the executor.

**File:** `.circleci/config.yml`

```yaml
version: 2.1
 
jobs:
  test:
    docker:
      # Use the official Microsoft image
      - image: mcr.microsoft.com/playwright:v1.43.0-jammy
    
    steps:
      - checkout
      
      # Install Node dependencies
      - run:
          name: Install dependencies
          command: npm ci
          
      # Execute the Playwright Suite
      - run:
          name: Run Playwright tests
          command: npx playwright test
          
      # Save the HTML report to CircleCI Artifacts
      - store_artifacts:
          path: playwright-report
          destination: playwright-report
          when: always # Ensure this runs on failure!
 
workflows:
  playwright-workflow:
    jobs:
      - test
```

### 3. The Golden Rule of CI Artifacts

Notice the pattern? In GitHub Actions we used `if: always()`, in GitLab we used `when: always`, and in CircleCI we used `when: always`. 

The most common mistake engineers make when setting up a new CI/CD pipeline is forgetting this flag. Without it, the CI runner will abort the moment `npx playwright test` throws an exit code of `1` (indicating a test failure). If the runner aborts, the artifact step never executes, and your developers are left with a failed build but no HTML report or Trace Viewer to debug it!

### Summary

Whether you use GitLab, CircleCI, Jenkins, or Bitbucket Pipelines, the core Playwright integration strategy remains identical:
1. Run the job inside the official `mcr.microsoft.com/playwright` Docker image.
2. Run `npm ci` and `npx playwright test`.
3. Unconditionally upload the `playwright-report` folder as a pipeline artifact.
