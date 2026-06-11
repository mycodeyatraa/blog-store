---
title: Configuration Management in Selenium Java: Dynamic Properties and Environment-Specific Config
date: 29-Jul-2024
lastUpdated: 11-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [selenium, java, configuration, properties, environment, configmanager, framework]
category: UI Automation
categories: [UI Automation, Selenium, Java]
excerpt: >-
  Eliminate hard-coded values from Selenium tests with a thread-safe ConfigManager in Java. Load base defaults and environment-specific overlays dynamically using system properties and .properties files.
readTime: 6 min read
---

﻿# Configuration Management in Selenium Java: Dynamic Properties and Environment-Specific Config

Hard-coded URLs, browser names, and timeouts in test classes are the fastest path to an unmaintainable framework. A **ConfigManager** decouples configuration from code — letting you switch environments with a single JVM flag (`-Denv=staging`) without touching a single test.

---

## What You Will Build

| Component | Purpose |
|---|---|
| `config.properties` | Base defaults (browser, waits, screenshot flag) |
| `qa.properties` | QA environment overlay |
| `staging.properties` | Staging environment overlay |
| `prod.properties` | Production environment overlay |
| `ConfigManager.java` | Thread-safe singleton — loads base + env overlay |
| `ConfigManagerTest.java` | TestNG validation of all ConfigManager behaviours |

---

## Configuration Loading Strategy

![diagram_1](https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/posts/configuration-management-selenium-java-configmanager/images/diagram_1.png)

---

## Project Structure

```
src/
 main/
  java/com/mycodeyatra/config/
   ConfigManager.java
  resources/
   config.properties          <- base defaults
 test/
  resources/environments/
   qa.properties              <- QA overlay
   staging.properties         <- Staging overlay
   prod.properties            <- Production overlay
  java/com/mycodeyatra/tests/
   ConfigManagerTest.java
```

---

## Step-by-Step Code Walkthrough

### 1. Base Config File (config.properties)

```
browser=chrome
headless=false
implicit.wait=10
explicit.wait=15
base.url=https://practice.mycodeyatra.com
screenshot.on.failure=true
```

### 2. Environment Overlay Files

#### qa.properties

```
base.url=https://practice.mycodeyatra.com
browser=chrome
headless=false
```

#### staging.properties

```
base.url=https://staging.mycodeyatra.com
browser=chrome
headless=true
```

#### prod.properties

```
base.url=https://mycodeyatra.com
browser=chrome
headless=true
```

### 3. ConfigManager (ConfigManager.java)

The core pattern: **double-checked locking** ensures thread-safety in parallel test runs, and **system property precedence** lets CI pipelines override any config value via `-Dkey=value`.

```java
package com.mycodeyatra.config;
import java.io.IOException;
import java.io.InputStream;
import java.util.Properties;
public class ConfigManager {
 private static final Properties PROPS = new Properties();
 private static volatile boolean initialised = false;
 private ConfigManager() {}
 public static void init() {
  if (initialised) return;
  synchronized (ConfigManager.class) {
   if (initialised) return;
   load("config.properties");
   String env = System.getProperty("env", "qa");
   load("environments/" + env + ".properties");
   initialised = true;
   System.out.println("[ConfigManager] Loaded config for env=" + env);
  }
 }
 private static void load(String resourcePath) {
  try (InputStream is = ConfigManager.class
   .getClassLoader().getResourceAsStream(resourcePath)) {
   if (is != null) {
    PROPS.load(is);
   } else {
    System.out.println("[ConfigManager] Resource not found (skipping): " + resourcePath);
   }
  } catch (IOException e) {
   throw new RuntimeException("[ConfigManager] Failed to load: " + resourcePath, e);
  }
 }
 public static String get(String key) {
  init();
  return System.getProperty(key, PROPS.getProperty(key, ""));
 }
 public static int getInt(String key, int defaultValue) {
  String val = get(key);
  try {
   return Integer.parseInt(val.trim());
  } catch (NumberFormatException e) {
   return defaultValue;
  }
 }
 public static boolean getBoolean(String key) {
  return Boolean.parseBoolean(get(key).trim());
 }
 public static void reload() {
  initialised = false;
  PROPS.clear();
  init();
 }
}
```

### 4. Validation Test (ConfigManagerTest.java)

```java
package com.mycodeyatra.tests;
import com.mycodeyatra.config.ConfigManager;
import org.testng.Assert;
import org.testng.annotations.BeforeClass;
import org.testng.annotations.Test;
public class ConfigManagerTest {
 @BeforeClass
 public void setup() {
  ConfigManager.reload();
 }
 @Test(description = "reads base.url from config")
 public void testBaseUrlLoaded() {
  String url = ConfigManager.get("base.url");
  System.out.println("[ConfigManager] base.url = " + url);
  Assert.assertFalse(url.isEmpty(), "base.url should not be empty");
  Assert.assertTrue(url.startsWith("http"), "base.url should be a valid URL");
 }
 @Test(description = "getInt returns correct timeout value")
 public void testGetInt() {
  int timeout = ConfigManager.getInt("explicit.wait", 10);
  System.out.println("[ConfigManager] explicit.wait = " + timeout);
  Assert.assertTrue(timeout > 0, "explicit.wait should be a positive integer");
 }
 @Test(description = "getBoolean parses headless flag correctly")
 public void testGetBoolean() {
  boolean headless = ConfigManager.getBoolean("headless");
  System.out.println("[ConfigManager] headless = " + headless);
  Assert.assertNotNull(String.valueOf(headless));
 }
 @Test(description = "missing key returns empty string, not null")
 public void testMissingKeyReturnsEmpty() {
  String val = ConfigManager.get("non.existent.key");
  Assert.assertEquals(val, "", "Missing key should return empty string");
  System.out.println("[ConfigManager] missing key = '" + val + "'");
 }
 @Test(description = "system property overrides config file value")
 public void testSystemPropertyOverride() {
  System.setProperty("browser", "firefox");
  String browser = ConfigManager.get("browser");
  Assert.assertEquals(browser, "firefox", "System property should override config file");
  System.clearProperty("browser");
  System.out.println("[ConfigManager] override test PASSED");
 }
}
```

---

## Switching Environments at Runtime

Run your entire test suite against any environment with a single JVM flag — no code changes:

```
# Run against QA (default)
mvn test
 
# Run against Staging
mvn test -Denv=staging
 
# Run against Production
mvn test -Denv=prod
 
# Override any single key inline
mvn test -Denv=staging -Dbrowser=firefox -Dheadless=true
```

---

## Key Takeaways

1. **Double-checked locking**: `volatile` + `synchronized` ensures `ConfigManager.init()` is thread-safe when parallel test threads race to initialise it.
2. **Layered config**: Base defaults in `config.properties` are overlaid by environment-specific files — properties defined in the overlay win.
3. **System property precedence**: `System.getProperty(key, fallback)` means any `-Dkey=value` JVM argument always wins over the file-based config — perfect for CI pipelines.
4. **Lazy initialisation**: Config is loaded on first call, not at class load time — so tests that never need config pay zero initialisation cost.
5. **`reload()` for test isolation**: Tests that need to verify different env configs can call `reload()` to force a fresh load.
