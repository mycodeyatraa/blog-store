---
title: When the Database is Blocked: Validating Data via GraphQL
date: 31-Jul-2025
lastUpdated: 31-Jul-2025
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [java, selenium, database-testing, graphql, restassured, api-testing, test-automation]
category: Selenium Java
categories: [Selenium Java, Database Testing]
excerpt: >-
  Direct database connections are often blocked by enterprise firewalls. Learn how to bypass the firewall by integrating RestAssured into Selenium Java to fire raw GraphQL queries and assert nested API responses.
readTime: 6 min read
---

# When the Database is Blocked: Validating Data via GraphQL

Throughout this series, we have assumed that your automation framework has direct network access to the backend database. We used JDBC to connect to MySQL and Postgres, and the Java Sync Driver to connect to MongoDB.

However, in massive Enterprise architectures, direct database connections are often completely blocked by aggressive firewall rules. To access the data, you must go through the API layer. 

While traditional REST APIs have been the industry standard for a decade, **GraphQL** is rapidly taking over. In this final tutorial of the Database Testing phase, we will learn how to bypass a blocked database and extract complex relational data directly using **RestAssured** and **GraphQL**!

---

## 1. Why GraphQL instead of REST?

In a traditional REST API architecture, if you want to validate a newly registered user, you might hit:
`GET /api/users/123`

But what if you also need to validate that their shipping address was created? You might have to hit a second endpoint:
`GET /api/users/123/addresses`

**GraphQL solves this by using a single endpoint (usually `/graphql`).** Instead of hitting multiple endpoints, you send a highly specific "Query String" defining exactly what data you want, and the GraphQL server returns a dynamic JSON payload matching your exact shape!

---

## 2. Integrating RestAssured

Because GraphQL operates over HTTP (usually via POST requests), we don't need a JDBC driver. Instead, we need an HTTP client. **RestAssured** is the undisputed champion of Java API testing.

Add RestAssured to your `pom.xml`:

```xml
<dependency>
    <groupId>io.rest-assured</groupId>
    <artifactId>rest-assured</artifactId>
    <version>5.4.0</version>
    <scope>test</scope>
</dependency>
```

---

## 3. Constructing the GraphQL Payload

A GraphQL request is simply an HTTP POST request where the body contains a specific JSON structure. The JSON must contain a `query` key, and optionally a `variables` key.

Let's construct a raw GraphQL query to fetch our newly registered user, their account status, and their nested shipping addresses:

```json
{
  "query": "query GetUserData($userEmail: String!) { user(email: $userEmail) { id accountStatus addresses { street city zip } } }",
  "variables": {
    "userEmail": "graphql_test@test.com"
  }
}
```

---

## 4. Writing the E2E GraphQL Test

Let's write a Selenium test that registers a user on the UI. Then, instead of connecting to a blocked Postgres database, we will use RestAssured to fire the GraphQL query, extract the nested JSON response, and assert that the data is correct!

```java
import io.restassured.RestAssured;
import io.restassured.path.json.JsonPath;
import io.restassured.response.Response;
import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.testng.Assert;
import org.testng.annotations.AfterMethod;
import org.testng.annotations.BeforeMethod;
import org.testng.annotations.Test;
import java.util.HashMap;
import java.util.Map;
public class GraphQLSeleniumTest {
    WebDriver driver;
    String graphqlEndpoint = "https://api.mycodeyatra.com/graphql";
    @BeforeMethod
    public void setup() {
        driver = new ChromeDriver();
    }
    @Test
    public void testRegistrationViaGraphQL() {
        String testEmail = "graphql_test@test.com";
        String testStreet = "789 API Blvd";
        // --- PART 1: The UI Workflow ---
        driver.get("https://practice.mycodeyatra.com/register");
        driver.findElement(By.id("email")).sendKeys(testEmail);
        driver.findElement(By.id("street")).sendKeys(testStreet);
        driver.findElement(By.id("submit-btn")).click();
        Assert.assertTrue(driver.findElement(By.id("success-banner")).isDisplayed());
        // --- PART 2: The GraphQL Validation ---
        // 1. Construct the GraphQL Query String
        String graphqlQuery = "query GetUserData($email: String!) { " +
                              "  user(email: $email) { " +
                              "    accountStatus " +
                              "    addresses { street city } " +
                              "  } " +
                              "}";
        // 2. Build the JSON Payload Map
        Map<String, Object> variables = new HashMap<>();
        variables.put("email", testEmail);
        Map<String, Object> graphqlPayload = new HashMap<>();
        graphqlPayload.put("query", graphqlQuery);
        graphqlPayload.put("variables", variables);
        // 3. Fire the POST Request using RestAssured
        Response response = RestAssured.given()
            .header("Content-Type", "application/json")
            .header("Authorization", "Bearer SUPER_SECRET_API_TOKEN") // If required
            .body(graphqlPayload)
            .post(graphqlEndpoint);
        // 4. Assert a 200 OK HTTP Status
        Assert.assertEquals(response.getStatusCode(), 200, "GraphQL API Failed!");
        // 5. Parse the JSON Response
        JsonPath jsonPath = response.jsonPath();
        // GraphQL responses always wrap the data in a "data" object!
        String accountStatus = jsonPath.getString("data.user.accountStatus");
        String dbStreet = jsonPath.getString("data.user.addresses[0].street");
        // 6. Assert the API Data Matches the UI Input!
        System.out.println("GraphQL returned status: " + accountStatus);
        Assert.assertEquals(accountStatus, "ACTIVE", "User should be active immediately!");
        Assert.assertEquals(dbStreet, testStreet, "Street address mismatch in backend!");
    }
    @AfterMethod
    public void teardown() {
        driver.quit();
    }
}
```

## Conclusion

When direct database access is blocked by security teams, API testing is your only escape hatch. While REST APIs require hitting multiple endpoints to piece together relational data, GraphQL allows you to construct massive, deeply nested data graphs in a single blazing-fast HTTP request!

By combining Selenium Java for frontend interaction with RestAssured and GraphQL for backend validation, you achieve true End-to-End coverage regardless of the firewall architecture.

**Congratulations!** You have officially completed the Database Testing module!

We have successfully bridged the gap between the UI and the Backend. We established JDBC connections to MySQL and Postgres, navigated NoSQL documents in MongoDB, and bypassed firewalls using GraphQL!

In our next and final chapter of this massive curriculum, we will step into **Phase 12: Continuous Integration**, where we will learn how to take all of our automated tests and run them in the cloud using Jenkins, Docker, and GitHub Actions!
