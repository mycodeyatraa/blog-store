---
title: Breaking the Relational Rules: NoSQL MongoDB Validation in Selenium
date: 29-Jul-2026
lastUpdated: 29-Jul-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [java, selenium, database-testing, mongodb, nosql, bson, test-automation]
category: Selenium Java
categories: [Selenium Java, Database Testing]
excerpt: >-
  Ditch JDBC and embrace Document architecture. Learn how to integrate the mongodb-driver-sync SDK to extract NoSQL BSON documents, navigate Collections, and assert nested JSON Arrays directly inside your Selenium Java tests.
readTime: 6 min read
---

# Breaking the Relational Rules: NoSQL MongoDB Validation in Selenium

In our previous tutorials, we leveraged JDBC to connect to relational databases like MySQL and PostgreSQL. We relied on structured tables, rigid columns, and standard SQL `SELECT` queries to validate our data.

But modern microservice architectures often abandon relational databases entirely in favor of **NoSQL** Document databases. The absolute undisputed king of NoSQL is **MongoDB**.

MongoDB does not have Tables or Rows. It has "Collections" and "Documents" stored in a flexible JSON-like format called BSON. Because it is not a relational database, you **cannot** use JDBC to connect to it!

In this tutorial, we will learn how to integrate the official `mongodb-driver-sync` SDK into our Selenium Java framework to execute powerful NoSQL Document assertions!

---

## 1. Adding the MongoDB Java Driver

Because JDBC is useless here, we must add the official MongoDB Java Sync Driver to our `pom.xml`.

```xml
<dependency>
    <groupId>org.mongodb</groupId>
    <artifactId>mongodb-driver-sync</artifactId>
    <version>4.11.1</version>
</dependency>
```

---

## 2. Understanding MongoDB Architecture

Before writing the code, you must understand the terminology mapping between SQL and MongoDB:

| SQL (Relational) | MongoDB (NoSQL) |
| :--- | :--- |
| Database | Database |
| Table | **Collection** |
| Row | **Document** (BSON/JSON format) |
| Column | **Field** |

A MongoDB connection string (URI) looks like this:
`mongodb://username:password@localhost:27017`

Notice that the Database name is *not* in the connection string. You connect to the Mongo Cluster, and then dynamically select the Database and Collection via Java code.

---

## 3. Writing the MongoDB End-to-End Test

Let's write a Selenium test that updates a user's shipping address on the UI. We will then connect to MongoDB, navigate to the `users` Collection, search for the Document matching our email, and assert that the address was correctly updated!

```java
import com.mongodb.client.MongoClient;
import com.mongodb.client.MongoClients;
import com.mongodb.client.MongoCollection;
import com.mongodb.client.MongoDatabase;
import org.bson.Document;
import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.testng.Assert;
import org.testng.annotations.AfterMethod;
import org.testng.annotations.BeforeMethod;
import org.testng.annotations.Test;
import static com.mongodb.client.model.Filters.eq;
public class MongoDBSeleniumTest {
    WebDriver driver;
    // MongoDB Connection URI
    String mongoUri = "mongodb://admin:secret@localhost:27017";
    @BeforeMethod
    public void setup() {
        driver = new ChromeDriver();
    }
    @Test
    public void testShippingAddressUpdatesInMongo() {
        String testEmail = "nosql_user@test.com";
        String newAddress = "123 Automation Lane";
        // --- PART 1: The UI Workflow ---
        driver.get("https://practice.mycodeyatra.com/profile");
        // Assume user is logged in. Update the shipping address.
        driver.findElement(By.id("shipping-address")).clear();
        driver.findElement(By.id("shipping-address")).sendKeys(newAddress);
        driver.findElement(By.id("save-profile-btn")).click();
        Assert.assertTrue(driver.findElement(By.id("success-toast")).isDisplayed());
        // --- PART 2: The MongoDB Validation ---
        // 1. Connect to the MongoDB Cluster (Using Try-With-Resources to Auto-Close)
        try (MongoClient mongoClient = MongoClients.create(mongoUri)) {
            // 2. Select the Database
            MongoDatabase database = mongoClient.getDatabase("ecommerce_db");
            // 3. Select the Collection (Table equivalent)
            MongoCollection<Document> collection = database.getCollection("users");
            // 4. Query the Collection for the specific User Document
            // Note: We use the `eq()` filter imported from com.mongodb.client.model.Filters
            Document userDocument = collection.find(eq("email", testEmail)).first();
            // 5. Assert the Document actually exists!
            Assert.assertNotNull(userDocument, "CRITICAL: User document not found in MongoDB!");
            // 6. Extract the fields from the BSON Document
            String dbName = userDocument.getString("name");
            String dbAddress = userDocument.getString("address");
            // 7. Assert the Database matches the UI input!
            System.out.println("Found Mongo Document for: " + dbName);
            Assert.assertEquals(dbAddress, newAddress, "MongoDB address did not update!");
            // 8. Asserting Nested JSON Arrays
            // MongoDB allows storing Arrays directly inside a document!
            java.util.List<String> roles = userDocument.getList("roles", String.class);
            Assert.assertTrue(roles.contains("CUSTOMER"), "User should have the CUSTOMER role!");
        } catch (Exception e) {
            Assert.fail("MongoDB validation failed: " + e.getMessage());
        }
    }
    @AfterMethod
    public void teardown() {
        driver.quit();
    }
}
```

---

## 4. Why MongoDB is Perfect for UI Automation

If you look closely at the Java code above, you'll notice how incredibly seamless it is. 

With JDBC/SQL, you have to write string-based SQL queries, execute a `Statement`, iterate through a `ResultSet`, and manually map columns. 

With the MongoDB Java Driver, the database effectively speaks "JSON/Java" natively. The `collection.find()` method directly returns a `org.bson.Document` object, which behaves exactly like a standard Java `Map<String, Object>`. You can pull Strings, Integers, and even massive nested `List` arrays directly out of the database document using standard getter methods!

## Conclusion

By breaking away from JDBC and utilizing the official `mongodb-driver-sync` SDK, you can seamlessly integrate NoSQL validation into your Selenium framework. 

However, connecting directly to raw databases is becoming increasingly rare in modern cloud-native enterprises. Many microservice architectures completely block direct database access for security reasons, forcing engineers to query data exclusively via APIs.

In our 5th and final tutorial of the Database Testing module, we will explore the cutting edge of data retrieval by combining Selenium Java with **GraphQL API Testing**!
