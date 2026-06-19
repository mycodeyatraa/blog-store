---
title: Querying the Truth: Connecting Selenium Java to MySQL
date: 25-Jul-2026
lastUpdated: 25-Jul-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [java, selenium, database-testing, mysql, jdbc, test-automation]
category: Selenium Java
categories: [Selenium Java, Database Testing]
excerpt: >-
  Stop relying on UI success banners. Learn how to import the mysql-connector-j driver, build JDBC connection strings, and write robust try-with-resources blocks to explicitly validate MySQL database records.
readTime: 6 min read
---

# Querying the Truth: Connecting Selenium Java to MySQL

In our introduction to End-to-End database testing, we learned the architectural importance of validating backend system states after a UI action completes. 

Now, it is time to write the code.

MySQL is one of the most widely used relational database management systems in the world. In this tutorial, we will learn how to install the official MySQL JDBC driver, establish a secure connection from our Java framework, execute a `SELECT` statement, and assert the results against our UI!

---

## 1. Adding the MySQL JDBC Driver

To allow Java to communicate with a MySQL database, we must add the official `mysql-connector-j` dependency to our `pom.xml`.

```xml
<dependency>
    <groupId>com.mysql</groupId>
    <artifactId>mysql-connector-j</artifactId>
    <version>8.2.0</version>
</dependency>
```

---

## 2. Understanding the JDBC Connection String

To establish a connection, the `DriverManager` requires a specific "Connection String" (also known as a JDBC URL). For MySQL, the syntax looks like this:

`jdbc:mysql://[host]:[port]/[database_name]`

For example, if your database is running locally on port 3306 and the database is named `ecommerce_db`:
`jdbc:mysql://localhost:3306/ecommerce_db`

*(Security Note: In an enterprise framework, you should never hardcode your database credentials. They should be passed dynamically via Environment Variables or a secrets manager like AWS Secrets Manager).*

---

## 3. Writing the E2E Registration Test

Let's write a complete End-to-End test. 

We will use Selenium to fill out a registration form on the UI. Then, instead of just trusting the "Success" banner, we will open a MySQL connection, query the `users` table, and explicitly assert that the data was saved correctly!

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
public class MySQLRegistrationTest {
    WebDriver driver;
    // Database Credentials
    String dbUrl = "jdbc:mysql://localhost:3306/ecommerce_db";
    String dbUser = "root";
    String dbPassword = "superSecretPassword";
    @BeforeMethod
    public void setup() {
        driver = new ChromeDriver();
    }
    @Test
    public void testUserRegistrationDataPersists() throws Exception {
        // --- PART 1: The UI Workflow ---
        driver.get("https://practice.mycodeyatra.com/register");
        String testEmail = "auto_user_" + System.currentTimeMillis() + "@test.com";
        String testPhone = "555-0198";
        driver.findElement(By.id("email")).sendKeys(testEmail);
        driver.findElement(By.id("phone")).sendKeys(testPhone);
        driver.findElement(By.id("password")).sendKeys("TestPass123!");
        driver.findElement(By.id("submit-btn")).click();
        // Assert the UI says success
        Assert.assertTrue(driver.findElement(By.id("success-banner")).isDisplayed());
        // --- PART 2: The Backend Validation ---
        // 1. Establish the Connection to MySQL
        Connection conn = DriverManager.getConnection(dbUrl, dbUser, dbPassword);
        // 2. Create the SQL Statement
        Statement stmt = conn.createStatement();
        String sqlQuery = "SELECT email, phone, account_status FROM users WHERE email = '" + testEmail + "'";
        // 3. Execute the Query and get the ResultSet
        ResultSet rs = stmt.executeQuery(sqlQuery);
        // 4. Assert that the record actually exists in the DB!
        Assert.assertTrue(rs.next(), "CRITICAL BUG: User was not saved to the MySQL database!");
        // 5. Extract the data columns
        String dbEmail = rs.getString("email");
        String dbPhone = rs.getString("phone");
        String dbStatus = rs.getString("account_status");
        // 6. Assert the DB data matches the UI inputs
        Assert.assertEquals(dbEmail, testEmail, "Email mismatch in DB!");
        Assert.assertEquals(dbPhone, testPhone, "Phone mismatch in DB!");
        // 7. Assert Business Logic (Default status should be 'ACTIVE')
        Assert.assertEquals(dbStatus, "ACTIVE", "New user should be ACTIVE in DB!");
        // 8. Close the DB Connections
        rs.close();
        stmt.close();
        conn.close();
    }
    @AfterMethod
    public void teardown() {
        driver.quit();
    }
}
```

---

## 4. Best Practices: Closing Connections

In the code above, we manually called `.close()` on the ResultSet, Statement, and Connection.

If your test fails on an assertion *before* reaching the `.close()` methods, the connection will remain open! If this happens 500 times during a CI/CD run, your database will throw a "Too Many Connections" error and crash.

**Always use a `try-with-resources` block or a `finally` block** to guarantee your connections are closed, regardless of whether the test passes or fails:

```java
try (Connection conn = DriverManager.getConnection(dbUrl, dbUser, dbPassword);
     Statement stmt = conn.createStatement();
     ResultSet rs = stmt.executeQuery(sqlQuery)) {
    Assert.assertTrue(rs.next());
    Assert.assertEquals(rs.getString("email"), testEmail);
} catch (Exception e) {
    Assert.fail("Database validation failed: " + e.getMessage());
}
// The try-with-resources block automatically closes rs, stmt, and conn!
```

## Conclusion

By executing a raw MySQL `SELECT` query via JDBC, we proved with 100% certainty that our UI registration workflow successfully persists data to the backend. 

In an enterprise environment, however, you will rarely connect to MySQL natively. Many large organizations rely heavily on **PostgreSQL**. In our next tutorial, we will learn how to swap out our MySQL driver for the Postgres driver and explore the nuances of Postgres schema architecture!
