---
title: "Building a CI/CD Pipeline for Selenium Kotlin with GitHub Actions"
date: "2025-03-10"
description: "Automate your testing lifecycle by integrating your Kotest Selenium framework with GitHub Actions. Learn how to trigger test runs on every pull request automatically!"
tags: ["Selenium", "Kotlin", "GitHub Actions", "CI/CD", "DevOps", "Automation"]
---

Welcome to Blog 33 of our Selenium Kotlin Mastery Series!

Writing amazing automated tests is only half the battle. If your tests only run when a developer manually triggers them from IntelliJ, they aren't truly "automated." True automation means your tests run independently, guarding your main codebase from regressions 24/7.

The industry standard way to achieve this is through a **Continuous Integration (CI) Pipeline**. In this post, we will build a CI pipeline using **GitHub Actions**, the native DevOps engine built directly into GitHub.

### Why GitHub Actions?

GitHub Actions allows you to define workflows using simple YAML files. Whenever an event occurs (like a developer pushing code or opening a Pull Request), GitHub spins up an isolated virtual machine, runs your Kotest framework, and reports the success or failure directly on the PR!

### Step 1: Creating the Workflow File

In the root directory of your project, create the following nested folders: `.github/workflows/`. Inside that folder, create a file named `selenium-tests.yml`.

```yaml
name: Selenium Kotlin CI
on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]
jobs:
  build-and-test:
    # Run the job on a fresh Ubuntu virtual machine hosted by GitHub
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
    - name: Run Selenium Tests (Headless)
      # We execute our Maven test goal here
      run: mvn test
```

### Step 2: Understanding the YAML

Let's break down exactly what this pipeline does:

1. **`on: push ... pull_request`**: These are the "triggers". The pipeline will run every time code is merged to `main` or a new PR targets `main`.
2. **`runs-on: ubuntu-latest`**: GitHub provides a free Linux VM. *Note: Ubuntu-latest comes with Google Chrome pre-installed by default!*
3. **`actions/setup-java@v3`**: This step installs Java 17 onto the blank VM and aggressively caches Maven dependencies so subsequent runs are lightning-fast.
4. **`run: mvn test`**: This is the final step where the magic happens. It executes our entire Kotest suite.

### Step 3: Headless Execution Requirement

Because GitHub Action VMs do not have graphical displays (monitors), all browsers must run in **Headless Mode**. 

If you remember from Phase 2, we built our `ThreadSafeDriverManager` to automatically pass `--headless=new` to Chrome Options. This architectural foresight means our framework is perfectly ready for CI/CD without changing a single line of Kotlin code!

### Execution Output on GitHub

When you commit this file and push to GitHub, navigate to the **Actions** tab of your repository. You will see a beautiful terminal output:

```
[INFO] Scanning for projects...
[INFO] Building mcyt-sel-kotlin 1.0-SNAPSHOT
...
[INFO] Running com.mycodeyatra.tests.Blog29_BehaviorSpecTest
Initializing new Headless Chrome Driver
...
[INFO] Tests run: 45, Failures: 0, Errors: 0, Skipped: 0
[INFO] BUILD SUCCESS
```

### Conclusion

Congratulations! You have now transitioned from a Test Automation Engineer to a true Software Development Engineer in Test (SDET). You have built a framework that automatically polices pull requests and blocks broken code from ever reaching production.

In our next blog, we will cover **Cross-Browser Parallel Testing in Kotest**, ensuring our UI looks flawless on Chrome, Firefox, and Edge simultaneously!
