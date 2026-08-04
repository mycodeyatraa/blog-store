---
title: Configuration Management - Playwright Java Design Patterns
date: 30-Jan-2026
lastUpdated: 30-Jan-2026
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
  Design type-safe configuration loaders reading config.properties, environment variables, and CLI overrides.
readTime: 9 min read
---

# Configuration Management - Playwright Java Design Patterns

Hardcoding URLs, credentials, or execution flags directly inside test scripts compromises security and prevents running tests across multiple environments (Dev, QA, Staging, Prod).

Configuration management provides a centralized mechanism to load properties dynamically based on active environment flags.

---

## 1. Configuration Hierarchy

```
       CLI Arguments (-Denv=qa -Dheadless=true)
                          |
                          v
         Environment Variables (SYS_ENV, BASE_URL)
                          |
                          v
     config.properties File (Default Fallback Properties)
```

---

## 2. Implementation Code Example

```java
// 1. Type-Safe Config Manager Implementation
public class ConfigManager {
    private static final Properties properties = new Properties();
 
    static {
        try (InputStream input = ConfigManager.class.getClassLoader().getResourceAsStream("config.properties")) {
            if (input != null) {
                properties.load(input);
            }
        } catch (IOException e) {
            throw new RuntimeException("Failed to load config.properties", e);
        }
    }
 
    public static String getProperty(String key, String defaultValue) {
        return System.getProperty(key, properties.getProperty(key, defaultValue));
    }
 
    public static String getBaseUrl() {
        return getProperty("baseUrl", "https://practice.mycodeyatra.com");
    }
}
```

---

## 3. Production Page Object (`src/main/java/com/mycodeyatra/pages/design/ConfigManagerPage.java`)

```java
package com.mycodeyatra.pages.design;
 
import com.microsoft.playwright.Page;
 
public class ConfigManagerPage {
    private final Page page;
 
    public ConfigManagerPage(Page page) {
        this.page = page;
    }
 
    public void navigateToConfiguredUrl(String url) {
        page.navigate(url);
    }
}
```

---

## 4. Key Takeaways

1. **System Property Priority**: Always check `System.getProperty(key)` first to allow Maven CLI overrides.
2. **Never Commit Passwords**: Load sensitive credentials via environment variables or secret vaults.

