---
title: Containerizing Automation: Running Selenium Grids with Docker
date: 17-Sep-2026
lastUpdated: 17-Sep-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [java, selenium, docker, docker-compose, selenium-grid, remote-webdriver, cicd, infrastructure]
category: Selenium Java
categories: [Selenium Java, Cloud & CI/CD]
excerpt: >-
  Solve the 'It works on my machine' problem forever. Learn how to abandon local ChromeDrivers and spin up isolated, highly scalable Selenium Grids using Docker Compose and RemoteWebDriver.
readTime: 6 min read
---

# Containerizing Automation: Running Selenium Grids with Docker

Welcome to the absolute final phase of this curriculum: **Phase 14 - Execution & CI/CD**. 

We have spent 13 phases building the perfect Selenium Java framework. We've mastered Page Objects, TestNG, APIs, Databases, Kafka events, and Observability. But right now, this multi-million dollar framework only runs on *your* laptop. 

If your laptop goes to sleep, the tests fail. If you run the tests on a junior engineer's laptop, they fail because they have the wrong version of Chrome installed. 

This is the classic "It works on my machine" problem. In this tutorial, we will solve it forever by containerizing our entire automation infrastructure using **Docker**.

---

## 1. What is Docker?

Docker is a platform that allows you to package an application (and all of its dependencies, like Chrome, Java, and Maven) into a standardized unit called a **Container**. 

Containers are isolated from the host machine. If you put your Selenium tests inside a Docker container, it does not matter if the host machine is running Windows 11, MacOS, or Ubuntu Linux. The container will always run exactly the same way.

---

## 2. The Problem with Local WebDriver

When you use `driver = new ChromeDriver()`, Selenium looks for the Chrome browser installed locally on your C: Drive or Mac Applications folder.

This creates massive CI/CD problems:
1. **Version Hell:** If your laptop has Chrome v130 but your Jenkins server has Chrome v128, your tests will fail on the server.
2. **Scalability:** You cannot run 50 Chrome browsers in parallel on a single Jenkins server without crashing the server's CPU.

The solution is the **Selenium Grid**. A Grid consists of a Hub (a router) and Nodes (machines that actually run browsers). Instead of installing Chrome on Jenkins, we will spin up a completely isolated, disposable Selenium Grid inside Docker!

---

## 3. Spinning Up a Dockerized Selenium Grid

Instead of manually installing Java, downloading Selenium Server JAR files, and configuring Hubs and Nodes via the command line, the Selenium HQ team provides official Docker Images that do everything for us!

We will use **Docker Compose** to instantly spin up a Selenium Grid with 1 Hub, 2 Chrome Nodes, and 1 Firefox Node.

Create a file named `docker-compose.yml` in the root of your project:

```yaml
version: "3"
services:
  # 1. The Hub (The Router)
  selenium-hub:
    image: selenium/hub:4.15.0
    container_name: selenium-hub
    ports:
      - "4444:4444" # Expose the Grid UI on port 4444
  # 2. Chrome Node
  chrome-node:
    image: selenium/node-chrome:4.15.0
    depends_on:
      - selenium-hub
    environment:
      - SE_EVENT_BUS_HOST=selenium-hub
      - SE_EVENT_BUS_PUBLISH_PORT=4442
      - SE_EVENT_BUS_SUBSCRIBE_PORT=4443
      - SE_NODE_MAX_SESSIONS=2
  # 3. Firefox Node
  firefox-node:
    image: selenium/node-firefox:4.15.0
    depends_on:
      - selenium-hub
    environment:
      - SE_EVENT_BUS_HOST=selenium-hub
      - SE_EVENT_BUS_PUBLISH_PORT=4442
      - SE_EVENT_BUS_SUBSCRIBE_PORT=4443
      - SE_NODE_MAX_SESSIONS=1
```

Open your terminal and run:

```bash
docker-compose up -d
```

Docker will automatically download the Selenium images, configure the network, and start the Grid! If you open `http://localhost:4444` in your browser, you will see the beautiful Selenium Grid UI showing your 2 Chrome instances and 1 Firefox instance, ready to receive tests.

---

## 4. Pointing Java to the Docker Grid

Now that our Grid is running in Docker, we need to tell our Java code to stop opening browsers on our local laptop, and instead send the commands over the network to the Docker Hub.

We do this using `RemoteWebDriver`.

Update your `BaseTest` class:

```java
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.remote.RemoteWebDriver;
import org.openqa.selenium.chrome.ChromeOptions;
import org.openqa.selenium.firefox.FirefoxOptions;
import java.net.URL;
public class BaseTest {
    protected ThreadLocal<WebDriver> driver = new ThreadLocal<>();
    public void setup(String browser) throws Exception {
        // The URL of our Docker Selenium Hub
        URL gridUrl = new URL("http://localhost:4444/wd/hub");
        if (browser.equalsIgnoreCase("chrome")) {
            ChromeOptions options = new ChromeOptions();
            // Send the request to Docker!
            driver.set(new RemoteWebDriver(gridUrl, options));
        } else if (browser.equalsIgnoreCase("firefox")) {
            FirefoxOptions options = new FirefoxOptions();
            driver.set(new RemoteWebDriver(gridUrl, options));
        }
    }
}
```

When you run your TestNG suite, Java will serialize your Selenium commands (`driver.get()`, `driver.findElement()`) and send them over HTTP to `localhost:4444`. The Docker Hub will instantly route the commands to one of the available Chrome/Firefox nodes inside the container network!

---

## 5. The Power of Scalability

Imagine your TestNG suite has grown to 500 tests. Running them sequentially takes 2 hours.

You want to run them in parallel. You update your `testng.xml` to `parallel="tests" thread-count="10"`. But your Docker Compose file only has 2 Chrome sessions available. The other 8 tests will be stuck waiting!

With Docker, you can scale your infrastructure with a single command. To instantly spawn 10 Chrome browsers:

```bash
docker-compose up --scale chrome-node=5 -d
```
*(Since `SE_NODE_MAX_SESSIONS=2`, 5 nodes * 2 sessions = 10 concurrent Chrome browsers!)*

You just scaled your enterprise testing infrastructure in 3 seconds without buying a single new laptop or VM!

## Conclusion

By migrating from local `ChromeDriver` instances to a Dockerized `RemoteWebDriver` Grid, you have eliminated the "It works on my machine" problem forever. 

Your automation framework is now decoupled from the underlying operating system. The browsers, the drivers, and the dependencies are all perfectly encapsulated in disposable Linux containers.

But wait—you still have to type `mvn clean test` on your laptop to send the commands to Docker. What if we want the tests to run automatically every time a developer pushes code to GitHub?

In our next tutorial, we will explore **GitHub Actions**, learning how to trigger our Dockerized TestNG suite automatically using YAML CI/CD pipelines!
