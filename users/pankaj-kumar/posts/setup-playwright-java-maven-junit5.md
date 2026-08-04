---
title: Setup Playwright + Java Project - Playwright Java Foundations
date: 04-Jan-2026
lastUpdated: 04-Jan-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [playwright, java, junit5, automation, testing, mycodeyatra]
category: Playwright Java Foundations
categories: [Playwright Java Foundations, Playwright Java, Test Automation]
excerpt: >-
  Complete step-by-step guide to configuring an enterprise Playwright Java repository with Apache Maven, JUnit 5, AssertJ, and Logback.
readTime: 10 min read
---

# Setup Playwright + Java Project - Playwright Java Foundations

Setting up a production-ready Playwright Java repository requires configuring Apache Maven, JUnit 5 Jupiter test runner, AssertJ fluent assertions, and SLF4J/Logback logging.

---

## 1. Directory Structure

Your Playwright Java framework should follow standard Maven multi-module architecture:

```
mcyt-plw-java/
├── pom.xml
├── src/
│   ├── main/
│   │   └── java/
│   │       └── com/mycodeyatra/
│   │           ├── config/
│   │           │   └── FrameworkConfig.java
│   │           └── pages/
│   │               └── BasePage.java
│   └── test/
│       ├── java/
│       │   └── com/mycodeyatra/
│       │       └── tests/
│       │           └── BaseTest.java
│       └── resources/
│           └── logback.xml
```

---

## 2. Complete Maven `pom.xml`

Below is the production-grade `pom.xml` configuration for Playwright Java:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
 
    <groupId>com.mycodeyatra</groupId>
    <artifactId>mcyt-plw-java</artifactId>
    <version>1.0-SNAPSHOT</version>
 
    <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <playwright.version>1.45.0</playwright.version>
        <junit.version>5.10.2</junit.version>
        <logback.version>1.5.3</logback.version>
    </properties>
 
    <dependencies>
        <!-- Playwright Java SDK -->
        <dependency>
            <groupId>com.microsoft.playwright</groupId>
            <artifactId>playwright</artifactId>
            <version>${playwright.version}</version>
        </dependency>
 
        <!-- JUnit 5 Engine -->
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter-api</artifactId>
            <version>${junit.version}</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter-engine</artifactId>
            <version>${junit.version}</version>
            <scope>test</scope>
        </dependency>
 
        <!-- Logging -->
        <dependency>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-classic</artifactId>
            <version>${logback.version}</version>
        </dependency>
    </dependencies>
 
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>3.2.5</version>
            </plugin>
        </plugins>
    </build>
</project>
```

---

## 3. Logback Logging Configuration (`src/test/resources/logback.xml`)

```xml
<configuration>
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>
 
    <root level="INFO">
        <appender-ref ref="STDOUT" />
    </root>
</configuration>
```

---

## 4. Running Tests from CLI

Execute tests across all browsers using Maven flags:

```bash
# Run all tests in headless mode
mvn test
 
# Run a specific test class
mvn test -Dtest=ProjectSetupTest
```

