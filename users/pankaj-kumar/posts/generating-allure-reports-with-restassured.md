---
title: Generating Allure Reports with RestAssured
date: 10-Feb-2025
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [restassured, java, allure, reporting, testng]
category: API Testing
categories: [API Testing, Automation, Java]
excerpt: >-
  Elevate your API automation framework by integrating Allure Reports with TestNG and RestAssured to automatically capture and display HTTP request and response payloads.
readTime: 6 min read
---

A robust test automation framework is only as good as its reporting mechanism. When tests fail, stakeholders and developers need to instantly see what went wrong, including the exact API requests and responses.

**Allure Reports** is the industry standard for creating visually stunning, highly detailed test reports. Fortunately, Allure provides a native adapter for RestAssured!

In this tutorial, we will learn how to integrate Allure into a RestAssured and TestNG framework.

---

## 1. Adding the Maven Dependencies

To generate Allure reports, you must first add the `allure-testng` and `allure-rest-assured` adapters to your `pom.xml`:

```xml
<!-- Allure TestNG Adapter -->
<dependency>
    <groupId>io.qameta.allure</groupId>
    <artifactId>allure-testng</artifactId>
    <version>2.27.0</version>
    <scope>test</scope>
</dependency>
<!-- Allure RestAssured Adapter -->
<dependency>
    <groupId>io.qameta.allure</groupId>
    <artifactId>allure-rest-assured</artifactId>
    <version>2.27.0</version>
</dependency>
```

*(Note: Ensure your `maven-surefire-plugin` is configured with the AspectJ weaver so TestNG can intercept the events!)*

## 2. Using the AllureRestAssured Filter

The magic of this integration lies in the `AllureRestAssured` filter. When you attach this filter to your RestAssured configuration, it automatically intercepts every HTTP request and response and attaches the raw HTTP payloads directly to the Allure Report UI!

We can attach this globally in our `@BeforeClass` setup:

```java
@Epic("Mock API Testing")
@Feature("User Management")
public class AllureReportingTest {
    @BeforeClass
    public void setup() {
        RestAssured.baseURI = "http://localhost:8080";
        RestAssured.basePath = "/api";
        // Globally attach the AllureRestAssured filter
        RestAssured.filters(new AllureRestAssured());
    }
    @Test(description = "Verify successful fetching of users list")
    @Description("This test validates that the /users endpoint returns a 200 OK status.")
    public void testGetUsersWithAllure() {
        Response response = executeGetUsersRequest();
        validateResponse(response, 200);
    }
    @Step("Execute GET /users request")
    private Response executeGetUsersRequest() {
        return RestAssured.given()
                .when()
                .get("/users")
                .then()
                .extract().response();
    }
    @Step("Validate that the status code is exactly {expectedStatusCode}")
    private void validateResponse(Response response, int expectedStatusCode) {
        Assert.assertEquals(response.getStatusCode(), expectedStatusCode);
    }
}
```

Notice the annotations we used:
* `@Epic` and `@Feature`: Groups tests together in the "Behaviors" tab of the report.
* `@Step`: Breaks your Java methods down into readable steps on the UI.
* `@Description`: Provides human-readable context.

## 3. Generating the Report

When you run your suite (`mvn clean test`), Allure will generate a folder named `allure-results` containing raw JSON files.

To render these files into a beautiful HTML dashboard, you simply run:

```bash
allure serve target/allure-results
```

*(Note: You must have the Allure command-line tool installed on your machine!)*

When the dashboard opens in your browser, you will see a detailed timeline. If you click on `testGetUsersWithAllure`, you will find a "Attachments" section that perfectly displays the raw HTTP Request (headers, cookies, URL) and the HTTP Response (status code, body).

This visibility is an absolute game-changer for debugging API failures in CI/CD pipelines!

In our final set of tutorials, we will explore Mocking, WireMock, and integrating this framework into Jenkins!
