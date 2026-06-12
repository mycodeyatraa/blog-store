---
title: Dynamic Security Testing: Integrating OWASP ZAP with Selenium
date: 19-Sep-2024
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [selenium, java, security-testing, owasp-zap, dast, vulnerability-scan]
category: UI Automation
categories: [UI Automation, Selenium, Java]
excerpt: >-
  Turn your automation framework into an active security scanner. Learn how to route Selenium traffic through OWASP ZAP and trigger dynamic vulnerability scans automatically via the Java API.
readTime: 5 min read
---

# Dynamic Security Testing: Integrating OWASP ZAP with Selenium

In our previous articles, we wrote targeted unit tests to assert the presence of security headers and mathematically validate JWTs. While those are fantastic foundational checks, they are entirely static.

To find critical vulnerabilities like SQL Injections, Cross-Site Scripting (XSS), and Broken Access Control, you need to actively attack the application while it's running. This is called **Dynamic Application Security Testing (DAST)**.

In this article, we will integrate **OWASP ZAP (Zed Attack Proxy)** directly into our Selenium Java framework to automatically scan for vulnerabilities every time our E2E tests run.

---

## 1. How OWASP ZAP Works with Selenium

OWASP ZAP acts as a "Man-in-the-Middle" proxy. 

Instead of Selenium's browser talking directly to your application's server, we configure the browser to route all its traffic through ZAP (usually running on `localhost:8080`). 

As your Selenium script naturally clicks buttons, fills forms, and navigates pages, ZAP passively records every single HTTP request and response. Once the functional test finishes, we use Java to trigger ZAP's API to launch an **Active Scan**—attacking all the URLs it just discovered—and generate an HTML report.

---

## 2. Configuring the Browser Proxy

The first step is telling Selenium to route its traffic through ZAP. Ensure you have the ZAP desktop application (or Docker container) running on port `8080`.

```java
package com.mycodeyatra.security;
 
import org.openqa.selenium.Proxy;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.chrome.ChromeOptions;
 
public class ZapProxyManager {
 
    public static WebDriver getProxiedDriver() {
 
        // Define the ZAP Proxy address
        String zapProxyAddress = "localhost:8080";
 
        Proxy proxy = new Proxy();
        proxy.setHttpProxy(zapProxyAddress);
        proxy.setSslProxy(zapProxyAddress);
 
        // Inject the proxy into ChromeOptions
        ChromeOptions options = new ChromeOptions();
        options.setProxy(proxy);
 
        // Ignore certificate errors because ZAP uses its own self-signed root cert
        options.setAcceptInsecureCerts(true);
 
        return new ChromeDriver(options);
    }
}
```

---

## 3. Controlling ZAP via the Java API

While you could just leave ZAP running in the background and click "Report" manually in the UI, true automation requires controlling ZAP programmatically. 

Add the official `zap-clientapi` to your `pom.xml`:

```xml
<dependency>
    <groupId>org.zaproxy</groupId>
    <artifactId>zap-clientapi</artifactId>
    <version>1.13.0</version>
</dependency>
```

Now, let's write our integrated Security Test:

```java
package com.mycodeyatra.security;
 
import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;
import org.testng.annotations.AfterClass;
import org.testng.annotations.BeforeClass;
import org.testng.annotations.Test;
import org.zaproxy.clientapi.core.ClientApi;
 
import java.io.File;
import java.nio.file.Files;
 
public class ZapSecurityIntegrationTest {
 
    private WebDriver driver;
 
    // Connect to ZAP running on localhost
    private ClientApi zapApi = new ClientApi("localhost", 8080, "YOUR_ZAP_API_KEY");
    private final String TARGET_URL = "https://mycodeyatra.com";
 
    @BeforeClass
    public void setup() {
        driver = ZapProxyManager.getProxiedDriver();
    }
 
    @Test
    public void executeFunctionalTestAndRecordTraffic() throws Exception {
        // 1. Functional execution - ZAP is passively recording this!
        driver.get(TARGET_URL + "/login");
        driver.findElement(By.id("username")).sendKeys("testuser");
        driver.findElement(By.id("password")).sendKeys("testpass");
        driver.findElement(By.id("loginBtn")).click();
 
        // Ensure page loads
        Thread.sleep(3000);
 
        // 2. Trigger ZAP Active Scan via API
        System.out.println("Spidering complete. Starting Active Scan...");
        zapApi.ascan.scan(TARGET_URL, "True", "False", null, null, null);
 
        // Polling to wait for scan to finish
        int progress;
        do {
            Thread.sleep(5000);
            progress = Integer.parseInt(zapApi.ascan.status(null).toString());
            System.out.println("Active Scan Progress: " + progress + "%");
        } while (progress < 100);
 
        System.out.println("Security Scan Complete!");
    }
 
    @AfterClass
    public void teardownAndGenerateReport() throws Exception {
        if (driver != null) {
            driver.quit();
        }
 
        // Retrieve HTML Report from ZAP API
        byte[] reportBytes = zapApi.core.htmlreport();
        File reportFile = new File("target/ZAP_Security_Report.html");
        Files.write(reportFile.toPath(), reportBytes);
 
        System.out.println("Vulnerability Report saved to: " + reportFile.getAbsolutePath());
    }
}
```

---

## System Architecture

Here is exactly how the data flows during a DAST execution cycle:

![diagram_1](https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/posts/dynamic-security-testing-integrating-owasp-zap-with-selenium/images/diagram_1.png)

## Critical Considerations

1. **NEVER run an Active Scan against Production.** Active scanning sends malicious payloads (SQL injections, drop table commands) and can corrupt databases or take down servers. Only run this against ephemeral CI environments.
2. **Execution Time:** Active scanning is slow. It can take 5 to 30 minutes depending on the size of the application. It is highly recommended to run security tests as a separate overnight job rather than blocking your main PR pipeline.

## Conclusion

By configuring Selenium to use OWASP ZAP as an upstream proxy, you completely eliminate the need to manually crawl your application for security testing. Your automated UI tests map out the application, and ZAP automatically attacks it, generating a comprehensive vulnerability report without any human intervention.

In our final security article, we will wrap up Phase 7 by looking at **Auth Security Patterns**, focusing on how to securely manage the credentials your automation framework uses so they don't leak into GitHub!
