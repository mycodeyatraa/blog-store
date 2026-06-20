---
title: "Behavior-Driven Development (BDD) in C#: Setting up Reqnroll & Feature Files"
date: "27-Jan-2025"
description: "Learn how to integrate BDD into your Selenium C# Framework using Reqnroll, the modern, official successor to SpecFlow, and write your first Gherkin Feature File."
categories: ["Selenium C#", "Test Automation", "BDD"]
tags: ["Selenium", "C#", ".NET", "BDD", "Reqnroll", "SpecFlow", "Gherkin", "Framework Design"]
author: "Pankaj Kumar"
lastUpdated: "27-Jan-2025"
---

Welcome to a brand new chapter in the **Selenium C# Mastery Series**! 

Up until now, we have built a powerful, code-heavy automation framework. However, non-technical stakeholders (like Product Managers or Business Analysts) cannot read C# code easily. They don't know what `Assert.That(driver.Title, Does.Contain("Home"));` means.

This is where **Behavior-Driven Development (BDD)** comes in. BDD allows you to write your tests in plain English using a syntax called **Gherkin**. 

For years, the standard tool for BDD in C# was **SpecFlow**. However, SpecFlow was recently discontinued. Today, we will be using its official, open-source successor: **Reqnroll**.

---

## 1. What is Reqnroll?

Reqnroll is a BDD framework for .NET. It reads plain English sentences (like "Given the user is on the login page") and binds them to your C# Selenium code!

It is fully compatible with NUnit and integrates perfectly with the architecture we have already built.

---

## 2. Installing Reqnroll Packages

To get started, open your terminal at the root of your test project and install the core Reqnroll packages. Because our framework is built on NUnit, we specifically need the NUnit adapter:

```sh
dotnet add package Reqnroll
dotnet add package Reqnroll.NUnit
```

Once installed, build your project to ensure there are no dependency conflicts:

```sh
dotnet build
```

---

## 3. Creating the Features Folder

BDD relies on **Feature Files**—text files with a `.feature` extension that contain your plain English scenarios.

Create a new folder in your project named `Features`.

---

## 4. Writing Your First Feature File

Inside the `Features` folder, create a new file named `Login.feature`.

We will use the **Gherkin Syntax**, which is built on the keywords: **Feature, Scenario, Given, When, Then, and And**.

Open `Login.feature` and paste the following:

```gherkin
Feature: User Authentication
  In order to access my secure account
  As a registered user
  I want to be able to log in to the application
  Scenario: Successful Login with Valid Credentials
    Given I navigate to the MyCodeYatra login page
    When I enter my username "admin@mycodeyatra.com"
    And I enter my password "AdminPass123!"
    And I click the login button
    Then I should see the dashboard page
```

### Understanding Gherkin
- **Feature**: A high-level description of the module being tested.
- **Scenario**: A specific test case within that module.
- **Given**: The initial state or precondition (e.g., navigating to a URL).
- **When / And**: The actions taken by the user.
- **Then**: The expected outcome or assertion.

---

## 5. Integrating with Visual Studio or VS Code

To get syntax highlighting and formatting for your `.feature` files:

- **Visual Studio**: Install the "Reqnroll for Visual Studio" extension from the marketplace.
- **VS Code**: Install the "Cucumber (Gherkin) Full Support" extension.

When you open `Login.feature`, you will notice that the steps might be underlined or highlighted with a warning. This is because we haven't written the underlying C# code to tell Reqnroll what to do when it reads "Given I navigate to the MyCodeYatra login page"!

---

## Conclusion

You have successfully installed Reqnroll and written your first plain-English Feature File using Gherkin syntax! 

Your Business Analysts can now write these files themselves without knowing a single line of C#!

However, these steps are currently "unbound." In our next blog, we will learn how to create **Step Definitions**, where we link these English sentences directly to our Selenium Page Object Models!

Happy Automating!
