---
id: "post-733"
title: "MySQL Testing with Playwright Java"
slug: "mysql-testing-with-playwright-java"
date: "16-Aug-2026"
author: "pankaj-kumar"
series: "playwright-java-enterprise-data"
seriesOrder: 2
topic: "2. MySQL Database Validation"
tags: ["Playwright", "Java", "MySQL", "JDBC", "Database"]
category: "Automation Testing"
categories: ["Automation Testing", "Playwright Java", "MySQL"]
excerpt: "Integrate MySQL Database verification into Playwright Java automation. Master transaction validation, stored procedure execution, and table assertions."
readTime: "8 min read"
---

# MySQL Testing with Playwright Java

MySQL is one of the most widely deployed relational database management systems in enterprise ecosystems. Coupling Playwright Java with MySQL JDBC drivers delivers robust backend state verification.

---

## 1. Architecture & Driver Setup

The Playwright Java framework uses MySQL Connector/J to connect to MySQL schemas, enabling pre-test setup, state assertions, and post-test data cleanup.

```
+------------------------------+
| Playwright Java Test Class   |
+------------------------------+
               |
               v (JDBC Driver: mysql-connector-j)
+------------------------------+
| MySQL Database Server        |
| - InnoDB Engine              |
| - Transactions / Constraints |
+------------------------------+
```

---

## 2. MySQL Connection Helper & Stored Procedure Utility

```java
// src/main/java/com/mycodeyatra/db/MySQLService.java
package com.mycodeyatra.db;
 
import java.sql.*;
import java.util.HashMap;
import java.util.Map;
 
public class MySQLService {
    private static final String URL = "jdbc:mysql://localhost:3306/shop_db?useUnicode=true&characterEncoding=UTF-8";
    private static final String USER = "mysql_admin";
    private static final String PASS = "AdminPass2026!";
 
    public static Map<String, Object> getOrderDetails(String orderId) {
        Map<String, Object> orderData = new HashMap<>();
        String sql = "SELECT order_id, total_amount, order_status FROM orders WHERE order_id = ?";
        
        try (Connection conn = DriverManager.getConnection(URL, USER, PASS);
             PreparedStatement pstmt = conn.prepareStatement(sql)) {
            
            pstmt.setString(1, orderId);
            try (ResultSet rs = pstmt.executeQuery()) {
                if (rs.next()) {
                    orderData.put("order_id", rs.getString("order_id"));
                    orderData.put("total_amount", rs.getDouble("total_amount"));
                    orderData.put("order_status", rs.getString("order_status"));
                }
            }
        } catch (SQLException e) {
            throw new RuntimeException("MySQL Query Error: " + e.getMessage(), e);
        }
        return orderData;
    }
 
    public static void executeCleanup(String customerId) {
        String sql = "DELETE FROM customers WHERE customer_id = ?";
        try (Connection conn = DriverManager.getConnection(URL, USER, PASS);
             PreparedStatement pstmt = conn.prepareStatement(sql)) {
            pstmt.setString(1, customerId);
            pstmt.executeUpdate();
        } catch (SQLException e) {
            System.err.println("Cleanup warning: " + e.getMessage());
        }
    }
}
```

---

## 3. Playwright MySQL Integration Test

```java
// src/test/java/com/mycodeyatra/tests/MySQLOrderValidationTest.java
package com.mycodeyatra.tests;
 
import com.microsoft.playwright.*;
import com.mycodeyatra.db.MySQLService;
import org.junit.jupiter.api.*;
import java.util.Map;
 
import static org.junit.jupiter.api.Assertions.*;
 
public class MySQLOrderValidationTest {
    private static Playwright playwright;
    private static Browser browser;
    private Page page;
 
    @BeforeAll
    static void setup() {
        playwright = Playwright.create();
        browser = playwright.chromium().launch(new BrowserType.LaunchOptions().setHeadless(true));
    }
 
    @BeforeEach
    void init() {
        page = browser.newPage();
    }
 
    @Test
    @DisplayName("Verify MySQL Order Record Creation on Checkout")
    void testOrderCreationInMySQL() {
        String orderId = "ORD-" + System.currentTimeMillis();
        
        // 1. Place order via Playwright UI
        page.navigate("https://mycodeyatra.com/checkout");
        page.fill("#order-ref", orderId);
        page.fill("#amount", "149.99");
        page.click("#place-order-btn");
 
        // 2. Assert UI success feedback
        assertTrue(page.isVisible("#order-confirmation"));
 
        // 3. Query MySQL Database for persistence validation
        Map<String, Object> dbRecord = MySQLService.getOrderDetails(orderId);
        assertFalse(dbRecord.isEmpty(), "Order record must exist in MySQL DB");
        assertEquals(149.99, (Double) dbRecord.get("total_amount"), 0.01);
        assertEquals("COMPLETED", dbRecord.get("order_status"));
    }
 
    @AfterAll
    static void tearDown() {
        if (browser != null) browser.close();
        if (playwright != null) playwright.close();
    }
}
```

---

## 4. Enterprise Best Practices & Comparison Summary

- **Parameterization**: Always use `PreparedStatement` to guard against SQL injection vulnerabilities in test suites.
- **Connection Isolation**: Maintain short-lived connections or static thread pools to avoid resource starvation during parallel test execution.
