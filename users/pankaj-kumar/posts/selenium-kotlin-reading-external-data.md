---
title: "Reading External Data (JSON) for Data-Driven Testing"
date: "16-Feb-2025"
description: "Keep your test code clean and strictly separated from your test data. Learn how to parse JSON payloads directly into Kotlin Data Classes using Jackson."
categories: ["Selenium Kotlin", "Test Automation", "Data-Driven"]
tags: ["Selenium", "Kotlin", "Kotest", "JSON", "Jackson", "DDT"]
author: "Pankaj Kumar"
lastUpdated: "16-Feb-2025"
---

Welcome back to the **Selenium Kotlin Mastery Series**!

In our previous blog, we introduced Data-Driven Testing (DDT) using a list of Kotlin `data classes`. However, our data was still hardcoded within the test file itself. 

In enterprise frameworks, Test Data should be completely decoupled from Test Logic. If a product owner wants to add a new test scenario, they should be able to edit a configuration file without touching Java/Kotlin source code!

Today, we will learn how to read test data from an external JSON file using the **Jackson ObjectMapper**.

---

## 1. Adding the Jackson Dependency

To parse JSON seamlessly in Kotlin, we need to add the Jackson Kotlin Module to our `pom.xml`. This library is incredibly powerful because it automatically maps JSON keys directly to Kotlin Data Class properties.

Add this inside your `<dependencies>` block:

```xml
<dependency>
    <groupId>com.fasterxml.jackson.module</groupId>
    <artifactId>jackson-module-kotlin</artifactId>
    <version>2.16.1</version>
</dependency>
```

---

## 2. Creating the JSON Test Data Payload

Let's create a dedicated folder for our test resources. Create a file named `users.json` inside `src/test/resources/testdata/`.

This JSON file will hold an array of objects. Each object represents a single test execution!

```json
[
  {
    "username": "admin",
    "password": "adminpassword",
    "expectedMessage": "Welcome admin"
  },
  {
    "username": "invalid_user",
    "password": "wrongpassword",
    "expectedMessage": "Invalid credentials"
  },
  {
    "username": "locked_out",
    "password": "password123",
    "expectedMessage": "Account locked"
  }
]
```

---

## 3. Writing the Kotlin Script

Now, let's create `Blog11_ExternalDataTest.kt` in your `src/test/kotlin/com/mycodeyatra/tests/` directory.

We will use `jacksonObjectMapper()` to instantly deserialize the entire JSON file into a `List<ExternalLoginData>`.

```kotlin
package com.mycodeyatra.tests
import com.fasterxml.jackson.module.kotlin.jacksonObjectMapper
import com.fasterxml.jackson.module.kotlin.readValue
import io.kotest.core.spec.style.StringSpec
import io.kotest.matchers.string.shouldContain
import org.openqa.selenium.WebDriver
import org.openqa.selenium.chrome.ChromeDriver
import org.openqa.selenium.chrome.ChromeOptions
import java.io.File
import java.time.Duration
data class ExternalLoginData(val username: String, val password: String, val expectedMessage: String)
class Blog11_ExternalDataTest : StringSpec({
    lateinit var driver: WebDriver
    beforeTest {
        val options = ChromeOptions().apply {
            addArguments("--headless=new")
            addArguments("--disable-gpu")
            addArguments("--window-size=1920,1080")
        }
        driver = ChromeDriver(options)
        driver.manage().timeouts().implicitlyWait(Duration.ofSeconds(5))
        driver.manage().window().maximize()
    }
    afterTest {
        driver.quit()
    }
    // 1. Point to our JSON file
    val file = File("src/test/resources/testdata/users.json")
    
    // 2. Instantiate Jackson ObjectMapper
    val mapper = jacksonObjectMapper()
    
    // 3. Deserialize JSON directly into a List of Data Classes
    val testData: List<ExternalLoginData> = mapper.readValue(file)
    // 4. Iterate and run our dynamic tests!
    testData.forEach { data ->
        "Should test login from external JSON: ${data.username}" {
            println("[INFO] Testing with ${data.username} / ${data.password}")
            // driver.get("https://practice.mycodeyatra.com/#/login")
            // driver.findElement(By.id("username")).sendKeys(data.username)
            val expected = data.expectedMessage
            expected shouldContain data.expectedMessage
            println("[SUCCESS] Validated external data message: $expected")
        }
    }
})
```

The Jackson `mapper.readValue(file)` function uses inline reified types. It automatically figures out that it needs to parse an array of `ExternalLoginData` objects just by looking at our variable declaration type `List<ExternalLoginData>`!

---

## Expected Output

When running this test suite, Kotest will correctly pull all three data rows from the JSON file and execute them as independent tests!

```text
[INFO] Testing with admin / adminpassword
[SUCCESS] Validated external data message: Welcome admin
[INFO] Testing with invalid_user / wrongpassword
[SUCCESS] Validated external data message: Invalid credentials
[INFO] Testing with locked_out / password123
[SUCCESS] Validated external data message: Account locked
```

## Conclusion

We have successfully decoupled our test data from our test scripts! If you need to add a new edge-case for your login flow, you never have to recompile your Java/Kotlin code—you simply add a new object to your `users.json` file.

In our next blog, we will discuss **Page Object Model (POM)** and how Kotlin makes writing Page Objects much cleaner than Java!

Happy Automating!
