---
title: "Setup Selenium + Kotlin Project (Maven & Kotest)"
date: "06-Feb-2025"
description: "Start your journey into modern Test Automation by setting up Selenium WebDriver with Kotlin, Maven, and the Kotest framework!"
categories: ["Selenium Kotlin", "Test Automation", "Setup"]
tags: ["Selenium", "Kotlin", "Maven", "Kotest", "JVM"]
author: "Pankaj Kumar"
lastUpdated: "06-Feb-2025"
---

Welcome to the **Selenium Kotlin Mastery** series! 

Kotlin is officially Google's preferred language for Android, and it has absolutely taken the JVM ecosystem by storm. It removes the boilerplate of Java, introduces null-safety by default, and provides incredibly powerful functional programming features. 

If you are a Java SDET looking to modernize your skill set, or a mobile engineer wanting to do web automation, Kotlin is the perfect choice.

In this first blog, we will set up a modern Kotlin automation project using **Maven** and **Kotest** (the idiomatic testing framework for Kotlin).

---

## 1. Prerequisites

Before we start, ensure you have the following installed on your machine:
- **Java Development Kit (JDK) 17** or higher
- **Maven** (3.8+)
- **IntelliJ IDEA** (Community or Ultimate) — JetBrains created Kotlin, so IntelliJ provides the absolute best experience.

---

## 2. Project Initialization

Create a new directory for your project and initialize a standard `pom.xml` file.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.mycodeyatra</groupId>
    <artifactId>mcyt-sel-kotlin</artifactId>
    <version>1.0-SNAPSHOT</version>
    <properties>
        <kotlin.version>1.9.22</kotlin.version>
        <selenium.version>4.18.1</selenium.version>
        <kotest.version>5.8.0</kotest.version>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
    </properties>
    <dependencies>
        <!-- Kotlin Standard Library -->
        <dependency>
            <groupId>org.jetbrains.kotlin</groupId>
            <artifactId>kotlin-stdlib</artifactId>
            <version>${kotlin.version}</version>
        </dependency>
        <!-- Selenium WebDriver -->
        <dependency>
            <groupId>org.seleniumhq.selenium</groupId>
            <artifactId>selenium-java</artifactId>
            <version>${selenium.version}</version>
        </dependency>
        <!-- Kotest Framework -->
        <dependency>
            <groupId>io.kotest</groupId>
            <artifactId>kotest-runner-junit5-jvm</artifactId>
            <version>${kotest.version}</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>io.kotest</groupId>
            <artifactId>kotest-assertions-core-jvm</artifactId>
            <version>${kotest.version}</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
    <build>
        <sourceDirectory>${project.basedir}/src/main/kotlin</sourceDirectory>
        <testSourceDirectory>${project.basedir}/src/test/kotlin</testSourceDirectory>
        <plugins>
            <!-- Kotlin Maven Plugin -->
            <plugin>
                <groupId>org.jetbrains.kotlin</groupId>
                <artifactId>kotlin-maven-plugin</artifactId>
                <version>${kotlin.version}</version>
                <executions>
                    <execution>
                        <id>compile</id>
                        <phase>compile</phase>
                        <goals><goal>compile</goal></goals>
                    </execution>
                    <execution>
                        <id>test-compile</id>
                        <phase>test-compile</phase>
                        <goals><goal>test-compile</goal></goals>
                    </execution>
                </executions>
                <configuration>
                    <jvmTarget>17</jvmTarget>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

### Why Kotest?
While you *can* use JUnit 5 or TestNG with Kotlin, **Kotest** is built from the ground up for Kotlin. It supports coroutines natively, has beautiful string-based test names, and includes fluent assertions out of the box!

---

## 3. Writing Your First Kotlin Test

In your project, create the following directory structure:
`src/test/kotlin/com/mycodeyatra/tests/`

Create a file named `Blog1_SetupTest.kt`:

```kotlin
package com.mycodeyatra.tests
import io.kotest.core.spec.style.StringSpec
import io.kotest.matchers.string.shouldContain
import org.openqa.selenium.WebDriver
import org.openqa.selenium.chrome.ChromeDriver
class Blog1_SetupTest : StringSpec({
    lateinit var driver: WebDriver
    // Runs before every test in this suite
    beforeTest {
        driver = ChromeDriver()
        driver.manage().window().maximize()
    }
    // Runs after every test
    afterTest {
        driver.quit()
    }
    // Kotest allows strings as test names! No @Test annotation required.
    "Should navigate to MyCodeYatra and verify title" {
        driver.get("https://mycodeyatra.com/")
        val title = driver.title
        println("Page Title: $title")
        // Kotest's elegant fluent assertion syntax
        title shouldContain "MyCodeYatra"
    }
})
```

### Key Kotlin Features Demonstrated:
1. `lateinit var`: In Kotlin, variables must be initialized. Since we initialize the driver in the `beforeTest` hook, we use `lateinit` to tell the compiler "I promise I will assign this before using it".
2. `StringSpec`: Kotest offers many styles (FunSpec, ShouldSpec, WordSpec). `StringSpec` is the simplest, allowing you to name tests with plain strings.
3. `shouldContain`: Notice how we didn't write `Assert.assertTrue(title.contains(...))`. Kotest's extension functions allow for incredibly readable, English-like assertions!

---

---

## Expected Output

When executing this test suite via Kotest, you will see the following output in your IntelliJ/Maven console confirming that the tests passed successfully:

```
[INFO] Running com.mycodeyatra.tests.Blog1_SetupTest

Blog1_SetupTest
  ✓ Should navigate to MyCodeYatra and verify title

1 tests completed, 1 successes, 0 failures, 0 ignored.
```

## Conclusion

Congratulations! You have successfully stepped into the modern era of JVM testing. With Kotlin and Kotest, your scripts will be significantly cleaner, shorter, and less prone to NullPointerExceptions.

In the next blog, we will dive into **Idiomatic Locators and Interactions**, exploring how to leverage Kotlin's unique features to make element interaction seamless!

Happy Automating!
