---
title: Scaling Visual Automation: Integrating BrowserStack Percy with Selenium Java
date: 05-Jul-2025
lastUpdated: 05-Jul-2025
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [java, selenium, visual-testing, percy, browserstack, dom-snapshot, test-automation]
category: Selenium Java
categories: [Selenium Java, Visual Testing]
excerpt: >-
  Integrate BrowserStack Percy into your Selenium framework. Learn how Percy captures DOM snapshots and renders them in the cloud for lightning-fast visual regressions and seamless GitHub PR review workflows.
readTime: 6 min read
---

# Scaling Visual Automation: Integrating BrowserStack Percy with Selenium Java

In our previous tutorials, we integrated Applitools Eyes to perform AI-powered visual validations. While Applitools is the undisputed leader in Visual AI, it is not the only enterprise player in the visual testing arena.

Another incredibly popular tool—especially for teams heavily invested in the BrowserStack ecosystem—is **Percy**.

Percy (acquired by BrowserStack) is a dedicated visual review platform designed to integrate seamlessly into CI/CD pipelines. Instead of taking full page screenshots locally, Percy captures the DOM snapshots and rendering assets from your Selenium tests, uploads them to the Percy cloud, and renders them in parallel across their massive device farm.

In this final tutorial of the Visual Testing series, we will learn how to integrate Percy into our Selenium Java framework!

---

## 1. How Percy Differs from Standard Screenshot Testing

When you run an open-source tool like `AShot`, your Java code physically commands the Chrome browser to take a `.png` screenshot. That screenshot is then saved to your hard drive and compared locally.

Percy works fundamentally differently:
1. When you call `percy.snapshot()`, Percy does **not** take a screenshot.
2. Instead, Percy serializes the entire HTML DOM structure of the page.
3. It intercepts all the CSS, Images, and Web Fonts currently loaded in the browser.
4. It bundles the DOM and the assets into a package and uploads it to the Percy Cloud via the `@percy/cli`.
5. Percy's cloud infrastructure re-constructs the web page using your exact DOM and CSS, and then renders it natively across Chrome, Firefox, Safari, and Edge in parallel!

Because Percy renders the UI in the cloud, your local Selenium tests run incredibly fast!

---

## 2. Setting Up the Project

To use Percy with Selenium Java, we need two components: the Java SDK and the NodeJS CLI.

**Step 1: Install the Percy CLI (NodeJS required)**

```bash
npm install -g @percy/cli
```

**Step 2: Add the Percy SDK to your `pom.xml`**

```xml
<dependency>
    <groupId>io.percy</groupId>
    <artifactId>percy-java-selenium</artifactId>
    <version>1.1.2</version>
</dependency>
```

**Step 3: Get your Percy Token**
Create a free account on BrowserStack Percy, create a new project, and retrieve your `PERCY_TOKEN`. Set this as an environment variable in your terminal.

---

## 3. Writing the Percy Selenium Test

Integrating Percy into your Selenium code is remarkably straightforward. 

```java
import io.percy.selenium.Percy;
import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.testng.annotations.AfterMethod;
import org.testng.annotations.BeforeMethod;
import org.testng.annotations.Test;
public class PercyVisualTest {
    WebDriver driver;
    Percy percy;
    @BeforeMethod
    public void setup() {
        driver = new ChromeDriver();
        // Initialize the Percy SDK
        percy = new Percy(driver);
    }
    @Test
    public void testCheckoutWorkflow() {
        // 1. Navigate to the application
        driver.get("https://practice.mycodeyatra.com/store");
        // 2. Take a Baseline Snapshot of the Homepage
        percy.snapshot("Storefront - Initial Load");
        // 3. Interact with the application
        driver.findElement(By.id("add-to-cart-shoes")).click();
        driver.findElement(By.id("checkout-btn")).click();
        // 4. Take another Snapshot of the Checkout Page
        // Percy will automatically upload the DOM to the cloud!
        percy.snapshot("Checkout Screen - Shoes Added");
    }
    @AfterMethod
    public void teardown() {
        driver.quit();
    }
}
```

---

## 4. Executing the Test via the Percy CLI

Here is the "catch" with Percy: Because it needs to securely package your DOM and upload it to the cloud, you cannot simply run your tests via your IDE's "Run" button or standard `mvn test`.

You must wrap your test execution command inside the `percy exec` CLI command. The CLI spins up a local proxy server that the Java SDK communicates with.

Open your terminal and run:

```bash
# Set your token
export PERCY_TOKEN="your_token_here"
# Execute your Maven tests inside the Percy wrapper!
npx percy exec -- mvn test -Dtest=PercyVisualTest
```

As the test runs, you will see the Percy CLI intercepting the DOM snapshots and uploading them. 

---

## 5. Reviewing the Results in the Percy Dashboard

Once the CLI finishes uploading, it will print a direct link to the Percy Dashboard in your terminal.

1. Click the link to open the dashboard.
2. Percy will display the UI differences side-by-side. 
3. If the differences are intentional (e.g., you changed the font color), click **"Approve"**. This instantly updates the Golden Baseline for future test runs!
4. If the differences are bugs, click **"Request Changes"** to fail the build.

This approval workflow is designed to integrate natively into GitHub Pull Requests, preventing any code from being merged until the Visual UI changes are approved by a human reviewer.

## Conclusion

Both Applitools and Percy offer incredible cloud-based visual testing capabilities. Applitools focuses heavily on Visual AI and Layout matching, while Percy excels in DOM serialization, deep GitHub integration, and a seamless developer approval workflow within the BrowserStack ecosystem.

By adding Percy to your Selenium framework, you guarantee that no CSS bug will ever slip into production unnoticed.

**Now** we have officially completed Phase 8! In our next phase, we dive into Accessibility Testing!
