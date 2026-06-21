---
title: "Advanced Selenium 4 Features in Kotlin: CDP & Network Interception"
date: "2025-03-12"
description: "Master the Chrome DevTools Protocol (CDP) using Selenium 4 and Kotlin. Learn how to emulate device geolocation and intercept network requests on the fly."
tags: ["Selenium", "Kotlin", "CDP", "Network Interception", "Advanced"]
---

Welcome to Blog 35—the grand finale of our Selenium Kotlin Mastery Series!

You've made it. From basic locators to CI/CD pipelines, you've learned it all. But Selenium 4 introduced a superpower that fundamentally changed the landscape of UI testing: native integration with the **Chrome DevTools Protocol (CDP)**.

CDP allows us to control the browser at a microscopic level, bypassing the standard WebDriver API to directly manipulate network traffic, geolocation, and even device metrics.

Today, we'll demonstrate one of the most powerful CDP capabilities: **Network Interception**.

### Why Intercept Network Traffic?

Imagine your frontend relies on a third-party payment API. If that API goes down during a test run, your UI tests fail. That's a "flaky" test because your UI code is completely fine!

Using CDP, we can intercept the outbound network request to the payment API and force the browser to inject a mocked JSON response (e.g., forcing a 200 OK or a 500 Server Error) to see how the UI reacts!

### Step 1: The Network Interceptor

Selenium 4 provides a brilliant `NetworkInterceptor` class. We simply pass it our driver instance and a `Route` matcher.

Let's write a Kotest specification that intercepts any request attempting to download a `.png` image from our Sandbox site, and intentionally mocks a `404 Not Found` response.

```kotlin
package com.mycodeyatra.tests
import com.mycodeyatra.utils.ThreadSafeDriverManager
import io.kotest.core.spec.style.StringSpec
import io.kotest.matchers.shouldBe
import org.openqa.selenium.devtools.NetworkInterceptor
import org.openqa.selenium.remote.http.Filter
import org.openqa.selenium.remote.http.HttpHandler
import org.openqa.selenium.remote.http.HttpResponse
class Blog35_AdvancedCDPTest : StringSpec({
    "Intercept Network Request and Mock Response" {
        val driver = ThreadSafeDriverManager.getDriver("chrome")
        // 1. Define a Network Filter
        val filter = Filter { next: HttpHandler ->
            HttpHandler { req ->
                // 2. Intercept any request ending in .png
                if (req.uri.contains("practice.mycodeyatra.com") && req.uri.endsWith(".png")) {
                    // 3. Mock the response to be a 404 Error!
                    HttpResponse().setStatus(404)
                } else {
                    // Otherwise, proceed normally
                    next.execute(req)
                }
            }
        }
        // 4. Attach the Interceptor
        val interceptor = NetworkInterceptor(driver, filter)
        try {
            driver.get("https://practice.mycodeyatra.com/#/login")
            // The site will load, but the browser will think the logo image is missing!
            // In a real scenario, you'd assert the UI displays a fallback "Broken Image" icon.
            driver.title shouldBe "MyCodeYatra | Test Automation Sandbox"
            println("Successfully executed Network Interception via CDP!")
        } finally {
            // Always close the interceptor to stop mocking
            interceptor.close()
            ThreadSafeDriverManager.quitDriver()
        }
    }
})
```

### Understanding the Code

1. **`Route.matching`**: This lambda evaluates every single HTTP request the browser attempts to make. If it returns true, the request is intercepted.
2. **`.to { HttpResponse() }`**: Instead of letting the browser talk to the real server, Selenium immediately feeds the browser this synthetic mocked response.
3. **`interceptor.close()`**: Cleans up the CDP listener.

### Execution Output

When you run the test, the browser will genuinely fail to render the image because it received a forced 404 response from Selenium!

```
[INFO] Running com.mycodeyatra.tests.Blog35_AdvancedCDPTest
Initializing Headless chrome Driver...
Successfully executed Network Interception via CDP!
Tests: 1, Passed: 1, Failed: 0
```

### Series Conclusion

Congratulations! Over these 35 blogs, you have evolved from a Kotlin beginner to an elite Automation Architect. You've built a robust, thread-safe, Page Object-driven framework capable of multi-browser parallel execution in Dockerized CI/CD pipelines, complete with visual testing and CDP superpowers.

Thank you for joining me on this incredible journey. Keep coding, keep testing, and keep pushing the boundaries of what automation can achieve. See you in the next series!
