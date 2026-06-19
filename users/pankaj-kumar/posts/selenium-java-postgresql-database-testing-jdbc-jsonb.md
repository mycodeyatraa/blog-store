---
title: Enterprise Relational Data: Integrating PostgreSQL with Selenium Java
date: 27-Jul-2026
lastUpdated: 27-Jul-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [java, selenium, database-testing, postgresql, jdbc, jsonb, test-automation]
category: Selenium Java
categories: [Selenium Java, Database Testing]
excerpt: >-
  Upgrade your database testing to Enterprise scale. Learn how to connect Selenium Java to PostgreSQL, navigate Postgres Schemas, and validate dynamic NoSQL-style JSONB columns using JDBC.
readTime: 6 min read
---

# Enterprise Relational Data: Integrating PostgreSQL with Selenium Java

In our previous tutorial, we successfully connected Selenium Java to MySQL. While MySQL is fantastic for smaller applications, massive enterprise systems—especially those relying heavily on geographic data, advanced JSON indexing, or complex analytical queries—almost universally rely on **PostgreSQL**.

PostgreSQL is arguably the most advanced open-source relational database in the world. 

In this tutorial, we will learn how to swap out our MySQL JDBC driver for Postgres, understand how Postgres schema architecture differs from MySQL, and execute an advanced End-to-End test that validates JSONB data columns!

---

## 1. Adding the PostgreSQL JDBC Driver

To allow Java to communicate with a Postgres database, we must remove the `mysql-connector-j` dependency and replace it with the official `postgresql` driver in our `pom.xml`.

```xml
<dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
    <version>42.7.1</version>
</dependency>
```

---

## 2. The PostgreSQL Connection String

The JDBC connection string for Postgres is very similar to MySQL, but with a different prefix:

`jdbc:postgresql://[host]:[port]/[database_name]`

Because Postgres uses port `5432` by default (instead of MySQL's `3306`), a local connection string looks like this:
`jdbc:postgresql://localhost:5432/ecommerce_db`

### Understanding Postgres Schemas
There is one major architectural difference between MySQL and Postgres. 
In MySQL, a "Database" is roughly equivalent to a "Schema". 
In Postgres, a Database contains **multiple** Schemas. The default schema is always named `public`. 

If your developers placed the `users` table inside a custom schema named `auth`, your SQL query must explicitly reference it:
`SELECT * FROM auth.users;`

---

## 3. Testing Advanced Postgres Features (JSONB)

One of the greatest features of PostgreSQL is its native support for **JSONB** (JSON Binary) columns. This allows developers to store NoSQL-style JSON documents directly inside a rigid relational table!

Imagine an e-commerce application where a user can save arbitrary "Preferences" (like dark mode, newsletter opt-in, or custom language). Instead of creating 50 separate columns in the database, the developers store all preferences in a single `JSONB` column named `user_settings`.

Let's write a Selenium test that toggles "Dark Mode" on the UI, connects to Postgres, and asserts that the JSONB column updated correctly!

```java
import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.testng.Assert;
import org.testng.annotations.AfterMethod;
import org.testng.annotations.BeforeMethod;
import org.testng.annotations.Test;
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.ResultSet;
import java.sql.Statement;
public class PostgresPreferencesTest {
    WebDriver driver;
    // Database Credentials
    String dbUrl = "jdbc:postgresql://localhost:5432/ecommerce_db";
    String dbUser = "postgres";
    String dbPassword = "superSecretPassword";
    @BeforeMethod
    public void setup() {
        driver = new ChromeDriver();
    }
    @Test
    public void testDarkModePreferenceSavesToPostgres() {
        // --- PART 1: The UI Workflow ---
        String userEmail = "existing_user@test.com";
        // 1. Log in and navigate to Settings
        driver.get("https://practice.mycodeyatra.com/login");
        driver.findElement(By.id("email")).sendKeys(userEmail);
        driver.findElement(By.id("password")).sendKeys("TestPass123!");
        driver.findElement(By.id("login-btn")).click();
        driver.findElement(By.id("nav-settings")).click();
        // 2. Toggle "Dark Mode" and Save
        driver.findElement(By.id("toggle-dark-mode")).click();
        driver.findElement(By.id("save-preferences")).click();
        // Assert the UI says success
        Assert.assertTrue(driver.findElement(By.id("success-toast")).isDisplayed());
        // --- PART 2: The PostgreSQL JSONB Validation ---
        String sqlQuery = "SELECT user_settings FROM public.users WHERE email = '" + userEmail + "'";
        try (Connection conn = DriverManager.getConnection(dbUrl, dbUser, dbPassword);
             Statement stmt = conn.createStatement();
             ResultSet rs = stmt.executeQuery(sqlQuery)) {
            Assert.assertTrue(rs.next(), "User record not found in Postgres!");
            // 3. Extract the JSONB column as a raw String
            String rawJsonSettings = rs.getString("user_settings");
            // Expected JSON: {"theme": "dark", "newsletter": true}
            System.out.println("Extracted JSONB: " + rawJsonSettings);
            // 4. Assert that the "theme" key inside the JSON was updated to "dark"
            Assert.assertTrue(rawJsonSettings.contains("\"theme\": \"dark\""), 
                "CRITICAL BUG: Dark mode was not saved to the Postgres JSONB column!");
        } catch (Exception e) {
            Assert.fail("Postgres validation failed: " + e.getMessage());
        }
    }
    @AfterMethod
    public void teardown() {
        driver.quit();
    }
}
```

---

## 4. Why Extracting JSONB is Powerful

In the example above, we extracted the JSONB column as a raw string and used `.contains()` to verify the update. 

If the JSON document is extremely large or complex, you can parse the extracted `rawJsonSettings` string using **Google Gson** or **Jackson** to convert it into a Java Object! This allows you to write strict, type-safe assertions against massive, deeply nested NoSQL-style data structures, all while remaining connected to a rigid PostgreSQL relational database.

## Conclusion

Switching your Selenium framework from MySQL to PostgreSQL is as simple as swapping the JDBC dependency and updating the connection string. However, understanding Postgres-specific features—like Schemas and JSONB columns—allows you to write significantly more powerful End-to-End tests.

While Postgres is great at handling JSON, it is still fundamentally a relational database. In our next tutorial, we will leave the relational world behind entirely and learn how to connect Selenium Java to the world's most popular NoSQL database: **MongoDB**!
