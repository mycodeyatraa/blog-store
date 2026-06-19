---
title: Automated Penetration Testing: Integrating OWASP ZAP with Selenium Java
date: 21-Jun-2026
lastUpdated: 21-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [java, selenium, devsecops, owasp-zap, security, penetration-testing, proxy]
category: Selenium Java
categories: [Selenium Java, Security]
excerpt: >-
  Turn your UI automation into a weaponized vulnerability scanner. Learn how to proxy your Selenium WebDriver traffic through OWASP ZAP to execute automated active security scans and generate HTML threat reports.
readTime: 7 min read
---

# Automated Penetration Testing: Integrating OWASP ZAP with Selenium Java

Throughout this security series, we have used native Selenium capabilities (CDP and WebStorage) to validate specific security configurations like CSP headers and JWT tokens. 

But what if you want to run a full, comprehensive vulnerability scan against your entire application automatically every time you run your UI tests?

Enter **OWASP ZAP** (Zed Attack Proxy). By integrating your Selenium Java test suite with OWASP ZAP, you can turn your functional UI tests into a weaponized vulnerability scanner that automatically detects SQL Injection, Cross-Site Scripting, and hundreds of other critical flaws.

---

## 1. How Does the Integration Work?

OWASP ZAP acts as an HTTP Proxy. Instead of Selenium sending traffic directly to your web server, we configure the ChromeDriver to route all its traffic *through* OWASP ZAP.

1. ZAP intercepts the request from Selenium.
2. ZAP passively analyzes the request and response for security flaws.
3. Once the Selenium test finishes navigating the site, we instruct ZAP (via its REST API) to run an **Active Scan**.
4. ZAP relentlessly attacks the specific pages that Selenium just navigated, looking for vulnerabilities.
5. Finally, we pull the vulnerability report and fail the test if high-risk flaws are found.

---

## 2. Setting Up OWASP ZAP

Before writing code, you need OWASP ZAP running on your machine.
1. Download and install OWASP ZAP from the official website.
2. Open ZAP and go to **Tools -> Options -> Local Proxies**.
3. Set the Address to `localhost` and the Port to `8080`.
4. Go to **Tools -> Options -> API** and configure an API Key (for this tutorial, we will disable the API key requirement for simplicity, but always use one in production).

---

## 3. Configuring Selenium to Route Through ZAP

We need to add the `zaproxy-client-api` to our `pom.xml` so we can communicate with ZAP programmatically:

```xml
<dependency>
    <groupId>org.zaproxy</groupId>
    <artifactId>zaproxy-client-api</artifactId>
    <version>1.13.0</version>
</dependency>
```

Now, let's configure the `ChromeDriver` to use the ZAP proxy:

```java
import org.openqa.selenium.Proxy;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.chrome.ChromeOptions;
import org.testng.annotations.BeforeMethod;
import org.zaproxy.clientapi.core.ClientApi;
public class ZapSecurityTest {
    ChromeDriver driver;
    ClientApi zapApi;
    String zapAddress = "127.0.0.1";
    int zapPort = 8080;
    String apiKey = ""; // Leave blank if API key is disabled in ZAP settings
    @BeforeMethod
    public void setup() {
        // 1. Initialize the ZAP API Client
        zapApi = new ClientApi(zapAddress, zapPort, apiKey);
        // 2. Create the Proxy Configuration
        Proxy proxy = new Proxy();
        proxy.setHttpProxy(zapAddress + ":" + zapPort);
        proxy.setSslProxy(zapAddress + ":" + zapPort);
        // 3. Attach Proxy to ChromeOptions
        ChromeOptions options = new ChromeOptions();
        options.setProxy(proxy);
        // **CRITICAL**: Bypass certificate errors since ZAP intercepts SSL traffic
        options.setAcceptInsecureCerts(true); 
        driver = new ChromeDriver(options);
    }
}
```

---

## 4. Writing the Automated Vulnerability Scan

Now we write a normal Selenium test. We log in, click some buttons, and fill out a form. 

Behind the scenes, ZAP is passively recording all of this traffic. Once the UI test finishes, we trigger the Active Scan!

```java
import org.testng.Assert;
import org.testng.annotations.AfterMethod;
import org.testng.annotations.Test;
import org.zaproxy.clientapi.core.ApiResponse;
import org.zaproxy.clientapi.core.ApiResponseElement;
@Test
public void executePenetrationTest() throws Exception {
    String targetUrl = "https://practice.mycodeyatra.com/";
    // 1. Perform normal UI Navigation
    // ZAP is proxying all of this and learning the site structure
    driver.get(targetUrl);
    driver.findElement(By.id("search")).sendKeys("Security Test");
    driver.findElement(By.id("search-btn")).click();
    // 2. Trigger ZAP Active Scan
    System.out.println("Initiating ZAP Active Scan...");
    ApiResponse resp = zapApi.ascan.scan(targetUrl, "True", "False", null, null, null);
    // The response contains the Scan ID
    String scanId = ((ApiResponseElement) resp).getValue();
    // 3. Poll until the scan is 100% complete
    int progress = 0;
    while (progress < 100) {
        Thread.sleep(5000);
        progress = Integer.parseInt(((ApiResponseElement) zapApi.ascan.status(scanId)).getValue());
        System.out.println("Active Scan Progress: " + progress + "%");
    }
    System.out.println("Active Scan Complete!");
    // 4. Validate the Results
    // We query ZAP to return any High or Medium risk vulnerabilities found
    byte[] reportBytes = zapApi.core.htmlreport();
    String reportHtml = new String(reportBytes);
    // We can save this report to a file for the security team
    java.nio.file.Files.write(java.nio.file.Paths.get("zap-security-report.html"), reportBytes);
    // Fail the build if ZAP found critical vulnerabilities!
    Assert.assertFalse(reportHtml.contains("High</risk>"), 
        "CRITICAL: ZAP found High-Risk Vulnerabilities! Check zap-security-report.html");
}
@AfterMethod
public void teardown() {
    if (driver != null) {
        driver.quit();
    }
}
```

---

## 5. Why the ZAP Integration is a Game Changer

Traditional security scanners simply crawl your website from the homepage. They often get stuck behind login screens, complex JavaScript menus, or multi-step checkout workflows.

By using Selenium to drive the browser, you effortlessly bypass the login screens and complex workflows, feeding the authenticated traffic directly into ZAP. ZAP then uses that authenticated context to execute devastatingly effective SQL Injection and XSS payloads deep inside your application.

## Conclusion

You have successfully built an automated, CI/CD-ready Penetration Testing framework using Selenium Java and OWASP ZAP. You are no longer just automating tests—you are actively securing your enterprise architecture.

In our final tutorial in this Security series, we will wrap up by exploring advanced **Auth Security Patterns** and defensive architecture!
