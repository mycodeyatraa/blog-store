---
title: RestAssured in CI/CD Pipelines
date: 15-Feb-2025
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [restassured, java, cicd, github-actions, automation]
category: API Testing
categories: [API Testing, Automation, CI/CD, Java]
excerpt: >-
  Learn how to integrate your automated RestAssured API test suites into GitHub Actions and Jenkins pipelines to guarantee production stability.
readTime: 6 min read
---

Writing automated API tests is only half the battle. The true value of test automation is unlocked when you integrate your tests directly into your **Continuous Integration / Continuous Deployment (CI/CD)** pipelines.

By running your RestAssured tests automatically on every `git push` or pull request, you ensure that no broken code is ever deployed to production.

In this tutorial, we will learn how to integrate a RestAssured TestNG suite into **GitHub Actions**.

---

## 1. Why Run API Tests in CI/CD?

When running tests locally on your machine, it's easy to miss edge cases or environment-specific configurations. Integrating RestAssured into a CI/CD pipeline guarantees:
1. **Consistency:** Tests run in a clean, isolated environment every single time.
2. **Immediate Feedback:** Developers are instantly notified if their commit breaks an API contract.
3. **Artifact Retention:** Test reports (like Allure) are automatically generated and hosted as artifacts for QA teams to review.

## 2. Setting Up GitHub Actions

Since RestAssured is fully integrated with Maven, running tests in CI is as simple as executing `mvn clean test`.

Create a file named `.github/workflows/restassured.yml` in the root of your project directory:

```yaml
name: RestAssured API Tests
on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
    - name: Checkout Code
      uses: actions/checkout@v3
    - name: Set up JDK 17
      uses: actions/setup-java@v3
      with:
        java-version: '17'
        distribution: 'temurin'
        cache: maven
    - name: Run RestAssured Tests
      run: mvn clean test
    - name: Upload Allure Results
      uses: actions/upload-artifact@v3
      if: always()
      with:
        name: allure-results
        path: target/allure-results
        retention-days: 5
```

## 3. How the Workflow Works

Let's break down the CI/CD configuration:

- **`on: push`**: The pipeline triggers automatically anytime a developer pushes code to the `main` branch.
- **`actions/setup-java@v3`**: This provisions an Ubuntu runner with Java 17, which matches our local `pom.xml` configurations.
- **`run: mvn clean test`**: This command downloads all dependencies (including RestAssured, TestNG, and Allure) and executes the entire suite.
- **`upload-artifact`**: Regardless of whether the tests Pass or Fail (`if: always()`), the pipeline bundles the raw `allure-results` directory and attaches it to the GitHub build log!

## 4. Viewing the Results

Once you commit this file to GitHub, navigate to the **Actions** tab in your repository. 

You will see the workflow kick off automatically. If an endpoint fails (perhaps your WireMock stub wasn't hit, or an Authentication token expired), the pipeline step will turn red, preventing the code from merging!

At the bottom of the workflow summary page, you can download the `allure-results` zip file, extract it locally, and run `allure serve` to view the beautiful HTML dashboard of your automated CI run.

In our next blog, we will graduate beyond basic functional testing and explore the fascinating world of **Observability-Driven Testing (ODT)**!
