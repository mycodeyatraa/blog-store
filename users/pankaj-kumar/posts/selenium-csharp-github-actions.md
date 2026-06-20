---
title: "Continuous Integration: Running Selenium C# Tests in GitHub Actions"
date: "26-Jan-2025"
description: "Take your framework to the cloud! Learn how to configure GitHub Actions to automatically run your entire Selenium C# test suite whenever developers push new code."
categories: ["Selenium C#", "Test Automation"]
tags: ["Selenium", "C#", ".NET", "CI/CD", "GitHub Actions", "DevOps", "Framework Design"]
author: "Pankaj Kumar"
lastUpdated: "26-Jan-2025"
---

Welcome back to the **Selenium C# Mastery Series**!

We have successfully built a massive, enterprise-grade test automation framework complete with Page Object Models, configuration managers, data-driven utilities, and beautiful ExtentReports.

However, an automation framework is completely useless if it only runs on your local laptop. The ultimate goal of automation is **Continuous Integration (CI)**: The tests should execute automatically in the cloud every single time a developer pushes new code, catching bugs before they reach production!

Today, we will achieve this using **GitHub Actions**.

---

## 1. What is GitHub Actions?

GitHub Actions is a CI/CD platform built directly into GitHub. It allows you to automate your software workflows. By writing a simple YAML configuration file, you can tell GitHub to launch a virtual machine (a "Runner"), download your code, install dependencies, and execute your Selenium tests automatically!

---

## 2. Creating the Workflow File

To use GitHub Actions, you don't need to install any software. You just need to create a specific folder structure in your repository.

In the root of your project, create a folder named `.github`. Inside it, create another folder named `workflows`. 

Finally, create a file named `selenium-tests.yml`.

```yml
name: Selenium C# Automation Suite
# When should these tests run?
on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]
  workflow_dispatch: # Allows you to run the tests manually from the UI!
jobs:
  run-tests:
    # Run the tests on a Windows Virtual Machine
    runs-on: windows-latest
    steps:
    # Step 1: Download your code from the repository
    - name: Checkout Code
      uses: actions/checkout@v4
    # Step 2: Install the .NET SDK so we can build the C# code
    - name: Setup .NET
      uses: actions/setup-dotnet@v4
      with:
        dotnet-version: '10.0.x' # Use your target framework version
    # Step 3: Download NuGet packages (Selenium, NUnit, ExtentReports, etc.)
    - name: Restore dependencies
      run: dotnet restore
    # Step 4: Build the project
    - name: Build Project
      run: dotnet build --no-restore
    # Step 5: Execute the tests!
    - name: Run Selenium Tests
      run: dotnet test --no-build --verbosity normal
```

---

## 3. Understanding the Magic

Let's break down exactly what this file does:

1. `on: push`: The second a developer pushes code to the `main` branch, a GitHub server wakes up and begins reading this file.
2. `runs-on: windows-latest`: GitHub gives you a free Windows Server Virtual Machine. We use Windows because it has browsers like Microsoft Edge and Chrome pre-installed, making it perfect for UI testing.
3. `dotnet restore & build & test`: We execute the exact same CLI commands you have been running locally on your own machine!

---

## 4. Uploading the Extent HTML Report

If the tests run on a server in the cloud, where does our `index.html` ExtentReport go? It gets destroyed when the server shuts down! 

To solve this, we must instruct GitHub Actions to upload the report as an **Artifact** before the server shuts down, so that QA Leads and Managers can download it!

Add this final step to your `.yml` file:

```yml
    # Step 6: Upload ExtentReports
    - name: Upload HTML Report
      uses: actions/upload-artifact@v4
      if: always() # Ensure the report uploads even if the tests fail!
      with:
        name: ExtentReports-Dashboard
        path: index.html # Path to where ExtentReports saves the file
```

Notice the `if: always()` condition? If a Selenium test fails, the `dotnet test` step throws an error, which normally cancels the rest of the workflow. `if: always()` ensures our report gets uploaded no matter what!

---

## 5. Seeing it in Action

Commit and push this `.yml` file to your GitHub repository.

1. Go to your GitHub Repository in your web browser.
2. Click the **Actions** tab at the top.
3. You will see your workflow actively running!
4. Click on it to watch the live terminal logs as the server executes your C# code.
5. Once complete, scroll to the bottom to download the **ExtentReports-Dashboard.zip** artifact!

---

## Conclusion

Congratulations! You have officially built a fully autonomous, enterprise-grade Selenium C# Framework. From raw code to cloud execution, you now possess the complete architecture required by top tech companies worldwide.

Thank you for following the **Selenium C# Mastery Series**. Keep coding, keep testing, and happy automating!
