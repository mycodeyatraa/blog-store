---
title: Logging Framework - Playwright Java Design Patterns
date: 31-Jan-2026
lastUpdated: 31-Jan-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [playwright, java, junit5, automation, design-patterns, mycodeyatra]
category: Playwright Java Design Patterns
categories: [Playwright Java Design Patterns, Playwright Java, Test Automation]
excerpt: >-
  Integrate SLF4J and Logback logging for colored terminal logs, file appenders, and Playwright action tracking.
readTime: 9 min read
---

# Logging Framework - Playwright Java Design Patterns

Structured logging is essential for diagnosing test failures in headless CI/CD runs. Replacing `System.out.println` statements with an enterprise logging facade like **SLF4J + Logback** provides timestamps, log levels (INFO, DEBUG, ERROR), and thread IDs.

---

## 1. Logback Console & File Appender Configuration

```xml
<configuration>
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d{{HH:mm:ss.SSS}} [%thread] %-5level %logger{{36}} - %msg%n</pattern>
        </encoder>
    </appender>
 
    <appender name="FILE" class="ch.qos.logback.core.FileAppender">
        <file>target/logs/test-execution.log</file>
        <append>false</append>
        <encoder>
            <pattern>%d{{yyyy-MM-dd HH:mm:ss.SSS}} [%thread] %-5level %logger{{36}} - %msg%n</pattern>
        </encoder>
    </appender>
 
    <root level="INFO">
        <appender-ref ref="CONSOLE" />
        <appender-ref ref="FILE" />
    </root>
</configuration>
```

---

## 2. Implementation Code Example

```java
// 1. Logger Usage in Test Class
public class LoggingTest {
    private static final Logger log = LoggerFactory.getLogger(LoggingTest.class);
 
    @Test
    void testLogging() {
        log.info("Navigating to MyCodeYatra Practice Sandbox...");
        page.navigate("https://practice.mycodeyatra.com/sandbox");
        log.info("Navigation successful!");
    }
}
```

---

## 3. Production Page Object (`src/main/java/com/mycodeyatra/pages/design/LoggingFrameworkPage.java`)

```java
package com.mycodeyatra.pages.design;
 
import com.microsoft.playwright.Page;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
 
public class LoggingFrameworkPage {
    private static final Logger log = LoggerFactory.getLogger(LoggingFrameworkPage.class);
    private final Page page;
 
    public LoggingFrameworkPage(Page page) {
        this.page = page;
    }
 
    public void navigateWithLogging() {
        log.info("Navigating to Sandbox");
        page.navigate("https://practice.mycodeyatra.com/sandbox");
    }
}
```

---

## 4. Key Takeaways

1. **Log Actionable Steps**: Log high-level user actions and navigation milestones.
2. **Logback Log Level Control**: Set log level to `DEBUG` when running local diagnostic builds.

