---
title: "Dockerizing Selenium Kotlin: Running Tests on Selenium Grid"
date: "2025-03-09"
description: "Learn how to decouple your test execution from your local machine by spinning up a Selenium Grid using Docker Compose and executing Kotest specs remotely."
tags: ["Selenium", "Kotlin", "Docker", "Grid", "CI/CD", "DevOps"]
---

Welcome to the 32nd post in our Selenium Kotlin Mastery Series!

Until now, every test we've written has executed on our local machine using local browser binaries. While this is great for development, it becomes a severe bottleneck when scaling. You can't tie up a developer's machine running 500 tests, nor can you rely on a CI/CD server natively having Chrome and Firefox installed.

The industry solution is **Docker**. By containerizing Selenium Grid, we can spin up disposable, identical browser environments in seconds. In this blog, we'll configure a `docker-compose.yml` and point our Kotlin framework to it!

### Step 1: The Docker Compose File

First, ensure Docker is installed on your machine. Then, create a `docker-compose.yml` file in the root of your project:

```yaml
version: "3"
services:
  selenium-hub:
    image: selenium/hub:4.18.1
    container_name: selenium-hub
    ports:
      - "4444:4444"
  chrome:
    image: selenium/node-chrome:4.18.1
    shm_size: 2gb
    depends_on:
      - selenium-hub
    environment:
      - SE_EVENT_BUS_HOST=selenium-hub
      - SE_EVENT_BUS_PUBLISH_PORT=4422
      - SE_EVENT_BUS_SUBSCRIBE_PORT=4423
```

To start your Grid, open your terminal and run:
`docker-compose up -d`

You can verify the Grid is running by visiting `http://localhost:4444` in your browser.

### Step 2: Modifying Kotest for Remote Execution

Instead of instantiating `ChromeDriver()` directly, we must instantiate a `RemoteWebDriver` and point it to the URL of our Docker Hub.

We will fetch the Hub URL dynamically using Kotlin's `System.getProperty()`. This allows us to pass the URL from the command line without hardcoding it.

```kotlin
package com.mycodeyatra.tests
import io.kotest.core.spec.style.StringSpec
import io.kotest.matchers.shouldBe
import org.openqa.selenium.Platform
import org.openqa.selenium.WebDriver
import org.openqa.selenium.chrome.ChromeOptions
import org.openqa.selenium.remote.DesiredCapabilities
import org.openqa.selenium.remote.RemoteWebDriver
import java.net.URL
class Blog32_DockerGridTest : StringSpec({
    "Execute Test on Remote Selenium Grid" {
        // Read hub URL from system properties
        val hubUrlString = System.getProperty("hubUrl", "http://localhost:4444/wd/hub")
        println("Connecting to Remote Grid at: $hubUrlString")
        // Define Capabilities for Docker (Linux)
        val chromeOptions = ChromeOptions()
        chromeOptions.setPlatformName(Platform.LINUX.name)
        // Instantiate the RemoteWebDriver
        val driver: WebDriver = RemoteWebDriver(URL(hubUrlString), chromeOptions)
        try {
            driver.get("https://practice.mycodeyatra.com/#/login")
            driver.title shouldBe "Practice Site"
            println("Successfully executed on Docker Grid!")
        } finally {
            driver.quit()
        }
    }
})
```

### Step 3: Execution via Maven

To trigger the execution from the command line and pass the System Property, use the `-D` flag:

```bash
mvn test -Dtest=Blog32_DockerGridTest -DhubUrl=http://localhost:4444/wd/hub
```

### Execution Output

```
[INFO] Running com.mycodeyatra.tests.Blog32_DockerGridTest
Connecting to Remote Grid at: http://localhost:4444/wd/hub
Successfully executed on Docker Grid!
Tests: 1, Passed: 1, Failed: 0
```

### Conclusion

By decoupling the test execution from your local machine, your framework is now ready to run in the cloud. You can easily scale your `docker-compose.yml` to spin up 50 parallel Chrome nodes and execute massive suites in minutes!

In our next blog, we will take the ultimate DevOps step: **Building a CI/CD Pipeline with GitHub Actions** to run our Kotest suite automatically on every code push!
