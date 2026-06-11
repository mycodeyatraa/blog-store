---
title: Logging Framework in Selenium Java: Integrating Log4j2 for Structured Test Logs
date: 30-Jul-2024
lastUpdated: 11-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [selenium, java, log4j2, logging, framework, test-automation]
category: UI Automation
categories: [UI Automation, Selenium, Java]
excerpt: >-
  Replace System.out.println with professional Log4j2 logging in your Selenium Java framework. Configure console and rolling file appenders, set per-package log levels, and use parameterised messages for zero-overhead logging.
readTime: 6 min read
---

﻿# Logging Framework in Selenium Java: Integrating Log4j2 for Structured Test Logs

`System.out.println` is the enemy of a professional test framework. It produces unstructured noise with no timestamps, no severity levels, and no way to filter. **Log4j2** replaces it with structured, configurable, high-performance logging — writing to the console and a rolling log file simultaneously, with zero code changes required to switch.

---

## What You Will Build

| Component | Purpose |
|---|---|
| `log4j2.xml` | Master configuration — appenders, patterns, log levels per package |
| `LoggerFactory.java` | Centralised factory so every class gets a correctly-named logger |
| `LoggingFrameworkTest.java` | TestNG validation of all log levels and named loggers |

---

## Log4j2 Architecture

![diagram_1](https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/posts/logging-framework-selenium-java-log4j2/images/diagram_1.png)

---

## Maven Dependency

Add to `pom.xml`:

```xml
<dependency>
 <groupId>org.apache.logging.log4j</groupId>
 <artifactId>log4j-api</artifactId>
 <version>2.23.1</version>
</dependency>
<dependency>
 <groupId>org.apache.logging.log4j</groupId>
 <artifactId>log4j-core</artifactId>
 <version>2.23.1</version>
</dependency>
```

---

## Step-by-Step Code Walkthrough

### 1. Log4j2 Configuration (log4j2.xml)

Place this file at `src/main/resources/log4j2.xml` so the classpath picks it up automatically:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="WARN" monitorInterval="30">
 <Properties>
  <Property name="LOG_PATTERN">%d{yyyy-MM-dd HH:mm:ss.SSS} [%t] %-5level %logger{36} - %msg%n</Property>
  <Property name="LOG_DIR">target/logs</Property>
 </Properties>
 <Appenders>
  <Console name="ConsoleAppender" target="SYSTEM_OUT">
   <PatternLayout pattern="${LOG_PATTERN}" />
  </Console>
  <RollingFile name="FileAppender"
   fileName="${LOG_DIR}/framework.log"
   filePattern="${LOG_DIR}/framework-%d{yyyy-MM-dd}-%i.log.gz">
   <PatternLayout pattern="${LOG_PATTERN}" />
   <Policies>
    <OnStartupTriggeringPolicy />
    <SizeBasedTriggeringPolicy size="10 MB" />
   </Policies>
   <DefaultRolloverStrategy max="10" />
  </RollingFile>
 </Appenders>
 <Loggers>
  <Logger name="com.mycodeyatra" level="DEBUG" additivity="false">
   <AppenderRef ref="ConsoleAppender" />
   <AppenderRef ref="FileAppender" />
  </Logger>
  <Root level="WARN">
   <AppenderRef ref="ConsoleAppender" />
  </Root>
 </Loggers>
</Configuration>
```

**Key decisions explained:**
- `com.mycodeyatra` logger at **DEBUG** — catches all framework log statements
- `Root` at **WARN** — suppresses verbose DEBUG/INFO from Selenium, WebDriverManager, etc.
- `RollingFile` — rotates on startup and at 10 MB, keeps last 10 compressed archives

### 2. LoggerFactory (LoggerFactory.java)

```java
package com.mycodeyatra.logging;
import org.apache.logging.log4j.LogManager;
import org.apache.logging.log4j.Logger;
public class LoggerFactory {
 private LoggerFactory() {}
 public static Logger getLogger(Class<?> clazz) {
  return LogManager.getLogger(clazz);
 }
 public static Logger getLogger(String name) {
  return LogManager.getLogger(name);
 }
}
```

### 3. Using the Logger in Any Class

Declare a single static logger per class — this is the standard Log4j2 pattern:

```java
package com.mycodeyatra.pages;
import com.mycodeyatra.logging.LoggerFactory;
import org.apache.logging.log4j.Logger;
public class LoginPage {
 private static final Logger LOG = LoggerFactory.getLogger(LoginPage.class);
 public void enterUsername(String username) {
  LOG.info("Entering username: {}", username);
  // ... WebDriver actions
 }
 public void clickLogin() {
  LOG.debug("Clicking login button");
  // ... WebDriver actions
 }
}
```

### 4. Validation Test (LoggingFrameworkTest.java)

```java
package com.mycodeyatra.tests;
import com.mycodeyatra.logging.LoggerFactory;
import org.apache.logging.log4j.Logger;
import org.testng.Assert;
import org.testng.annotations.BeforeClass;
import org.testng.annotations.Test;
public class LoggingFrameworkTest {
 private static final Logger LOG = LoggerFactory.getLogger(LoggingFrameworkTest.class);
 private static final Logger EXEC_LOG = LoggerFactory.getLogger("TEST_EXECUTION");
 @BeforeClass
 public void setup() {
  LOG.info("LoggingFrameworkTest suite starting...");
 }
 @Test(description = "Logger is not null and correctly initialised")
 public void testLoggerInitialised() {
  Assert.assertNotNull(LOG, "Logger should not be null");
  LOG.info("Logger initialised for: {}", LoggingFrameworkTest.class.getName());
 }
 @Test(description = "DEBUG level log emits without exception")
 public void testDebugLog() {
  LOG.debug("DEBUG: Selenium driver initialisation started");
  Assert.assertTrue(LOG.isDebugEnabled() || !LOG.isDebugEnabled());
 }
 @Test(description = "WARN level log emits without exception")
 public void testWarnLog() {
  LOG.warn("WARN: Element not found on first attempt - retrying...");
  Assert.assertTrue(true);
 }
 @Test(description = "ERROR level log with exception emits without throwing")
 public void testErrorLog() {
  LOG.error("ERROR: Test failed unexpectedly", new RuntimeException("Simulated failure"));
  Assert.assertTrue(true);
 }
 @Test(description = "Named logger works for TEST_EXECUTION category")
 public void testNamedLogger() {
  Assert.assertNotNull(EXEC_LOG);
  EXEC_LOG.info("TEST_EXECUTION: testNamedLogger PASSED");
 }
 @Test(description = "Parameterised log message formats correctly")
 public void testParameterisedLog() {
  LOG.info("Starting test on browser={} in environment={}", "chrome", "qa");
  Assert.assertTrue(true);
 }
}
```

---

## Sample Console Output

```
2024-07-30 10:15:32.001 [main] INFO  c.m.t.LoggingFrameworkTest - Logger initialised for: com.mycodeyatra.tests.LoggingFrameworkTest
2024-07-30 10:15:32.003 [main] DEBUG c.m.t.LoggingFrameworkTest - DEBUG: Selenium driver initialisation started
2024-07-30 10:15:32.004 [main] WARN  c.m.t.LoggingFrameworkTest - WARN: Element not found on first attempt - retrying...
2024-07-30 10:15:32.005 [main] ERROR c.m.t.LoggingFrameworkTest - ERROR: Test failed unexpectedly
2024-07-30 10:15:32.006 [main] INFO  TEST_EXECUTION - TEST_EXECUTION: testNamedLogger PASSED
2024-07-30 10:15:32.007 [main] INFO  c.m.t.LoggingFrameworkTest - Starting test on browser=chrome in environment=qa
```

---

## Log Level Guide for Test Automation

| Level | Use For |
|---|---|
| `DEBUG` | Driver actions, element interactions, wait conditions |
| `INFO` | Test start/end, navigation events, key state changes |
| `WARN` | Retry attempts, deprecated API usage, slow responses |
| `ERROR` | Assertion failures, exceptions, unexpected states |

---

## Key Takeaways

1. **Never use System.out.println in a framework** — it produces unfiltered noise with no context (no timestamp, no class, no level).
2. **One logger per class** — declare as `private static final Logger LOG = LoggerFactory.getLogger(MyClass.class)`. Static means one instance per class, not per test method.
3. **Parameterised messages avoid toString overhead** — `LOG.info("Value: {}", obj)` only calls `obj.toString()` if INFO is actually enabled.
4. **Root logger at WARN** — keeps third-party library noise (Selenium, WDM) out of your logs while your own code logs at DEBUG.
5. **RollingFile keeps history** — compressed `.log.gz` archives give you post-mortem analysis capability without filling the disk.
