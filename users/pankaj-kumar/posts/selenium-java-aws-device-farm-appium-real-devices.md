---
title: Breaking out of Docker: Running Tests on AWS Device Farm
date: 26-Sep-2026
lastUpdated: 26-Sep-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [java, selenium, aws, device-farm, appium, mobile-testing, cicd, cloud]
category: Selenium Java
categories: [Selenium Java, Cloud & CI/CD]
excerpt: >-
  You can't run an iPhone inside a Docker container. Learn how to securely authenticate with the AWS SDK and route your RemoteWebDriver commands directly to physical mobile devices sitting in an Amazon data center.
readTime: 6 min read
---

# Breaking out of Docker: Running Tests on AWS Device Farm

In the previous tutorials, we containerized our Selenium Grid using Docker. This works perfectly when you need to test your web application on Google Chrome or Mozilla Firefox running on a Linux server.

But what if your company builds a mobile-first application? What if your CEO demands that the login page must be tested on a physical iPhone 15 Pro running iOS 17, and a physical Samsung Galaxy S24 running Android 14?

You cannot run an iPhone inside a Docker container. You could buy physical phones and plug them into your Jenkins server via USB cables, but managing the batteries, OS updates, and wifi connections of a "Local Device Lab" is a nightmare.

Enter the **Cloud Device Lab**. In this tutorial, we will learn how to route our Appium and Selenium tests to real, physical smartphones sitting in an Amazon data center using **AWS Device Farm**.

---

## 1. What is AWS Device Farm?

AWS Device Farm is a cloud-based service that houses thousands of real smartphones, tablets, and desktop browsers. 

Instead of routing our `RemoteWebDriver` commands to `localhost:4444` (our Docker Grid), we will dynamically request a physical device from Amazon via their API, receive a temporary secure URL, and route our WebDriver commands directly to that physical device in the cloud!

---

## 2. Setting up the AWS SDK

To interact with AWS, we need to import the official Java SDK into our `pom.xml`:

```xml
<dependency>
    <groupId>software.amazon.awssdk</groupId>
    <artifactId>devicefarm</artifactId>
    <version>2.20.100</version>
</dependency>
```

You must also configure your AWS IAM credentials on your machine or CI/CD server using the standard `~/.aws/credentials` file, ensuring the user has the `AWSDeviceFarmFullAccess` policy.

---

## 3. Generating the Temporary Grid URL

Amazon does not give you a static URL for their Selenium Grid. Because you are renting a physical device by the minute, you must call their API to generate a temporary, securely signed URL that expires after a set amount of time.

Let's write a utility class to fetch this URL:

```java
import software.amazon.awssdk.regions.Region;
import software.amazon.awssdk.services.devicefarm.DeviceFarmClient;
import software.amazon.awssdk.services.devicefarm.model.CreateTestGridUrlRequest;
import software.amazon.awssdk.services.devicefarm.model.CreateTestGridUrlResponse;
public class AWSDeviceFarmUtil {
    public static String getGridUrl() {
        // Create the AWS Client (Requires IAM credentials to be configured)
        DeviceFarmClient client = DeviceFarmClient.builder()
                .region(Region.US_WEST_2) // Device Farm is strictly hosted in Oregon
                .build();
        // Request a temporary URL valid for 300 seconds (5 minutes)
        CreateTestGridUrlRequest request = CreateTestGridUrlRequest.builder()
                .expiresInSeconds(300)
                .projectArn("arn:aws:devicefarm:us-west-2:1234567890:testgrid-project:YOUR-PROJECT-ID")
                .build();
        CreateTestGridUrlResponse response = client.createTestGridUrl(request);
        System.out.println("AWS Device Farm URL Generated!");
        return response.url();
    }
}
```

---

## 4. Pointing RemoteWebDriver to AWS

Now that we have the securely signed URL, we simply update our `BaseTest` to request a specific mobile browser (or desktop browser) via `DesiredCapabilities`.

Let's request a mobile Safari browser running on an iPhone:

```java
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.remote.DesiredCapabilities;
import org.openqa.selenium.remote.RemoteWebDriver;
import java.net.URL;
public class BaseTest {
    protected ThreadLocal<WebDriver> driver = new ThreadLocal<>();
    public void setupMobileWeb() throws Exception {
        // 1. Get the temporary AWS Grid URL
        String awsUrl = AWSDeviceFarmUtil.getGridUrl();
        // 2. Define the exact device we want Amazon to plug in for us
        DesiredCapabilities caps = new DesiredCapabilities();
        caps.setCapability("browserName", "safari");
        caps.setCapability("platformName", "iOS");
        caps.setCapability("appium:deviceName", "iPhone 15 Pro");
        caps.setCapability("appium:automationName", "XCUITest");
        // 3. Connect!
        driver.set(new RemoteWebDriver(new URL(awsUrl), caps));
        System.out.println("Successfully connected to a physical iPhone in AWS!");
    }
}
```

When you run this code, Amazon will physically allocate an iPhone 15 Pro in their data center, install the Appium WebDriverAgent, open Safari, and execute your TestNG script!

---

## 5. Integrating with CI/CD

To run this in Jenkins or GitHub Actions, you do not need to spin up a Docker Grid (`docker-compose up`). Your infrastructure is entirely outsourced to Amazon.

You simply need to ensure your CI/CD pipeline injects the AWS Credentials securely so the Java SDK can authenticate:

```yaml
# GitHub Actions Example
jobs:
  run-aws-tests:
    runs-on: ubuntu-latest
    steps:
    - name: Checkout Code
      uses: actions/checkout@v3
    # Inject IAM Credentials via GitHub Secrets
    - name: Configure AWS Credentials
      uses: aws-actions/configure-aws-credentials@v2
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-region: us-west-2
    - name: Run Tests on Device Farm
      run: mvn clean test -Dsuite=mobile-web
```

## Conclusion

By integrating AWS Device Farm (or similar services like BrowserStack and SauceLabs), you have completely eliminated infrastructure maintenance from your QA team's responsibilities.

You don't need to buy physical phones. You don't need to write Docker Compose files. You simply inject an API key into your CI/CD pipeline, and you instantly have access to thousands of perfectly maintained physical devices on demand.

**This is the peak of Enterprise Test Execution.**

We have now learned everything. From the absolute basics of `driver.get()` in Phase 1, to Docker Grids, ELK logging, and AWS physical device farms in Phase 14. 

There is only one topic left. In our next and final tutorial of the entire Selenium Java curriculum, we will summarize everything into the **Phase 14 Capstone: The Ultimate CI/CD Strategy**.
