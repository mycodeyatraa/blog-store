---
title: Network Monitoring - Playwright Java Core UI
date: 20-Jan-2026
lastUpdated: 20-Jan-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [playwright, java, junit5, automation, ui-automation, mycodeyatra]
category: Playwright Java Core UI
categories: [Playwright Java Core UI, Playwright Java, Test Automation]
excerpt: >-
  Intercept HTTP requests, mock REST API payloads, abort slow assets, and inspect response status codes with page.route().
readTime: 10 min read
---

# Network Monitoring - Playwright Java Core UI

Modern UI applications depend heavily on backend REST APIs. Testing edge cases (such as 500 Server Errors, 401 Unauthorized timeouts, or slow network responses) in live environments can be challenging.

Playwright Java provides built-in network interception APIs via `page.route()`.

---

## 1. Network Interception Use Cases

```
                                 Page Request Sent
                                        |
                                        v
                       +----------------------------------+
                       |     Playwright page.route()      |
                       +----------------------------------+
                        /               |                \
                       v                v                 v
                route.fulfill()   route.continue()   route.abort()
               (Mock JSON Body)   (Modify Headers)  (Block Images/Ads)
```

---

## 2. Network Interception Examples

### 1. Mock REST API Response (`route.fulfill()`)
```java
page.route("**/api/users", route -> {
    route.fulfill(new Route.FulfillOptions()
        .setStatus(200)
        .setContentType("application/json")
        .setBody("{"users": [{"id": 1, "name": "Mocked User"}]}"));
});
```

### 2. Block Heavy Image Assets for High-Speed Runs (`route.abort()`)
```java
page.route("**/*.{png,jpg,jpeg,svg}", Route::abort);
```

---

## 3. Production Page Object (`src/main/java/com/mycodeyatra/pages/NetworkMonitoringPage.java`)

```java
package com.mycodeyatra.pages;
 
import com.microsoft.playwright.Page;
import com.microsoft.playwright.Route;
 
public class NetworkMonitoringPage {
    private final Page page;
 
    public NetworkMonitoringPage(Page page) {
        this.page = page;
    }
 
    public void navigateAndMock() {
        page.route("**/api/config", route -> {
            route.fulfill(new Route.FulfillOptions()
                .setStatus(200)
                .setBody("{"featureEnabled": true}"));
        });
        page.navigate("https://practice.mycodeyatra.com/widgets");
    }
}
```

---

## 4. Key Takeaways

1. **Mock Early**: Attach `page.route()` handlers prior to executing navigation or button triggers.
2. **Clean Mock Teardown**: Unroute handlers via `page.unroute()` when finished.

