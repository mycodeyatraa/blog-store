---
title: Enterprise CI/CD: Translating Pipelines to GitLab and Azure DevOps
date: 23-Sep-2026
lastUpdated: 23-Sep-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [java, selenium, gitlab, azure-devops, ado, cicd, pipeline, yaml, enterprise]
category: Selenium Java
categories: [Selenium Java, Cloud & CI/CD]
excerpt: >-
  A Senior SDET isn't tied to a single tool. Learn how to translate the core concepts of Continuous Integration into GitLab CI docker-in-docker syntax and Azure DevOps Pipeline tasks.
readTime: 6 min read
---

# Enterprise CI/CD: Translating Pipelines to GitLab and Azure DevOps

In our previous tutorial, we automated our Selenium execution using GitHub Actions. GitHub Actions is fantastic for open-source projects and modern startups.

However, if you work at a massive Fortune 500 bank, a healthcare provider, or a government agency, you probably aren't using GitHub. Large enterprises overwhelmingly rely on two major platforms for their internal code repositories and CI/CD pipelines: **GitLab CI** and **Azure DevOps (ADO)**.

The good news? The underlying logic of Continuous Integration never changes. If you understand how to write a YAML pipeline for GitHub Actions, you already know how to write one for GitLab or Azure. You just need to learn the syntax translation.

In this tutorial, we will translate our Dockerized Selenium pipeline into both GitLab CI and Azure DevOps formats!

---

## 1. The Core CI/CD Concepts

No matter which tool you use, every CI/CD pipeline consists of the same 5 core steps:

1. **Trigger:** When should this pipeline run? (e.g., Push to `main`).
2. **Environment:** Where should it run? (e.g., Ubuntu Linux VM).
3. **Setup:** Install dependencies (e.g., JDK 17, Docker).
4. **Execution:** Run the tests (e.g., `mvn clean test`).
5. **Artifacts:** Save the results (e.g., Upload Allure JSON).

Let's see how these map to different platforms.

---

## 2. GitLab CI (`.gitlab-ci.yml`)

GitLab uses a file named `.gitlab-ci.yml` placed in the root of your repository. 

Unlike GitHub Actions which provisions a full VM, GitLab relies heavily on **Docker Runners**. Instead of "installing" Java on a Linux machine, you simply tell GitLab to execute your pipeline *inside* a pre-built Java Docker container!

```yaml
# 1. Define the stages of your pipeline
stages:
  - test
# 2. Define the job
selenium-regression:
  stage: test
  # 3. Environment: Run inside a pre-built Maven/Java Docker image!
  image: maven:3.8.4-openjdk-17
  # Services: GitLab's way of spinning up parallel Docker containers (like Docker Compose)
  services:
    - name: selenium/standalone-chrome:4.15.0
      alias: selenium-grid # We can access this via http://selenium-grid:4444
  # 4. Execution
  script:
    - echo "Running Selenium Tests..."
    # We pass the Grid URL as a System Property so Java knows where to route tests
    - mvn clean test -DsuiteXmlFile=testng.xml -Dgrid.url=http://selenium-grid:4444/wd/hub
  # 5. Artifacts
  artifacts:
    when: always # Run even if tests fail
    paths:
      - target/allure-results/
    expire_in: 1 week
```

### Key Differences in GitLab:
*   Notice the `services` keyword. Instead of writing a manual `docker-compose.yml` file, GitLab natively supports spinning up sidecar containers (like the Selenium Chrome Standalone image) right next to your test code!
*   Because your tests run inside the `maven` container and the browser runs inside the `selenium` container, you must pass the `alias` (`http://selenium-grid:4444`) to your Java code so it knows where to send the WebDriver commands over the internal GitLab Docker network.

---

## 3. Azure DevOps (`azure-pipelines.yml`)

Microsoft Azure DevOps (ADO) uses a file named `azure-pipelines.yml`. 

ADO syntax is much closer to GitHub Actions. It provisions a Virtual Machine (an "Agent Pool") and runs scripts sequentially.

```yaml
# 1. Trigger
trigger:
  - main
# 2. Environment
pool:
  vmImage: 'ubuntu-latest'
# 3. Steps
steps:
# Step A: Install Java
- task: JavaToolInstaller@0
  inputs:
    versionSpec: '17'
    jdkArchitectureOption: 'x64'
    jdkSourceOption: 'PreInstalled'
# Step B: Spin up Docker Compose
- script: |
    docker-compose up -d
    sleep 15
  displayName: 'Start Docker Selenium Grid'
# Step C: Execute Maven
- task: Maven@4
  inputs:
    mavenPomFile: 'pom.xml'
    goals: 'clean test'
    options: '-DsuiteXmlFile=testng.xml'
    publishJUnitResults: true # ADO natively parses standard TestNG/JUnit XML!
    testResultsFiles: '**/surefire-reports/TEST-*.xml'
  displayName: 'Run Selenium Suite'
# Step D: Upload Allure Artifacts
- task: PublishPipelineArtifact@1
  condition: always()
  inputs:
    targetPath: '$(System.DefaultWorkingDirectory)/target/allure-results'
    artifact: 'AllureResults'
    publishLocation: 'pipeline'
```

### Key Differences in Azure DevOps:
*   ADO relies heavily on specialized "Tasks" (`task: Maven@4`) rather than raw bash scripts (`run: mvn clean test`). These Tasks have built-in logic to parse TestNG XML results and automatically display them natively in the Azure DevOps Dashboard without requiring a third-party Allure plugin!

## Conclusion

A Senior SDET is not tied to a single tool. A Senior SDET understands the underlying architecture.

Whether your company uses GitHub Actions, GitLab CI, Azure DevOps, or Jenkins, the goal remains the same: Spin up an isolated environment, inject your code, route commands to a Dockerized Grid, execute Maven, and extract the reporting artifacts. 

If you understand the workflow, learning the YAML syntax of a new platform takes less than an afternoon.

We have successfully run our tests in the cloud. But we are still relying on Docker containers running *Chrome* and *Firefox*. 

What if your company builds a mobile application? You can't run an iPhone inside a Docker container. In our next tutorial, we will explore **AWS Device Farm**, learning how to route our Appium/Selenium tests to real, physical mobile devices sitting in an Amazon data center!
