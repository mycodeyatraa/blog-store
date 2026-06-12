---
title: Contract Testing: OpenAPI Validation with RestAssured
date: 18-Aug-2024
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [selenium, java, restassured, api-testing, contract-testing, openapi, swagger]
category: UI Automation
categories: [UI Automation, Selenium, Java]
excerpt: >-
  Shift your API validation left with Contract Testing. Learn how to automatically intercept RestAssured requests and validate them against OpenAPI Swagger definitions.
readTime: 5 min read
---

# Contract Testing: OpenAPI Validation with RestAssured

In modern microservices, the backend API is essentially a contract between the server and the client (such as your UI). If the backend developer changes a field name from `firstName` to `first_name` without warning, your UI will break, and your E2E tests will fail. 

While functional UI tests catch this eventually, they are slow and fail late in the pipeline. **Contract Testing** shifts this validation left. By strictly validating API responses against their OpenAPI (Swagger) specifications using RestAssured, we can catch breaking API changes the exact second they happen.

---

## 1. Functional API Testing vs Contract Testing

**Functional API Testing:** 
You send a POST request to `/users` and assert that the response status is 201, and the returned user ID is not null.

**Contract Testing:** 
You send a POST request to `/users` and assert that the entire response schema, headers, and payload strictly adhere to the `swagger.yaml` or `openapi.json` definition defined by the architecture team.

![diagram_1](https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/posts/contract-testing-openapi-validation-with-restassured/images/diagram_1.png)

---

## 2. Setting Up Swagger Request Validator

To implement OpenAPI contract testing in RestAssured, we use the excellent `swagger-request-validator-restassured` library by Atlassian.

Add this dependency to your `pom.xml`:

```xml
<dependency>
    <groupId>com.atlassian.oai</groupId>
    <artifactId>swagger-request-validator-restassured</artifactId>
    <version>2.39.0</version>
</dependency>
```

---

## 3. Creating the Contract Validation Filter

RestAssured allows us to inject "Filters" into our `RequestSpecification`. We can create a filter that automatically intercepts every API response and validates it against our OpenAPI spec.

```java
package com.mycodeyatra.api.filters;
 
import com.atlassian.oai.validator.restassured.OpenApiValidationFilter;
import io.restassured.filter.Filter;
 
public class ContractValidator {
 
    private static final String SWAGGER_URL = "https://api.mycodeyatra.com/v2/api-docs";
 
    /**
     * Returns a RestAssured Filter that automatically validates 
     * requests and responses against the OpenAPI specification.
     */
    public static Filter getOpenApiFilter() {
        return new OpenApiValidationFilter(SWAGGER_URL);
    }
}
```

---

## 4. Applying the Contract Test in Your Hybrid Framework

Now, we can integrate this filter directly into our existing `ApiClient` base specification. This ensures that every single API call made during our Hybrid tests implicitly acts as a contract test!

```java
package com.mycodeyatra.api;
 
import com.mycodeyatra.api.filters.ContractValidator;
import io.restassured.RestAssured;
import io.restassured.specification.RequestSpecification;
 
public class ApiClient {
 
    public static RequestSpecification getBaseSpec() {
        return RestAssured.given()
                .baseUri("https://api.mycodeyatra.com")
                .header("Content-Type", "application/json")
                .header("Accept", "application/json")
                // Apply the OpenAPI Contract Validator globally
                .filter(ContractValidator.getOpenApiFilter());
    }
}
```

### Writing the Test

The beauty of this architecture is that your actual test code does not change at all. The validation happens automatically in the background.

```java
package com.mycodeyatra.tests;
 
import com.mycodeyatra.api.ApiClient;
import org.testng.annotations.Test;
 
public class ContractTest {
 
    @Test
    public void verifyUserEndpointContract() {
 
        // The OpenApiValidationFilter will automatically throw an exception 
        // if the response payload or headers violate the swagger.yaml definition.
        ApiClient.getBaseSpec()
                .get("/v1/users/123")
                .then()
                .statusCode(200);
    }
}
```

---

## Why This is a Game Changer

1. **Shift Left:** Breaking changes in the API payload are caught instantly during the API testing phase, long before the UI tests even execute.
2. **Zero Maintenance Assertions:** You no longer need to write 50 lines of `Assert.assertEquals()` to verify every field in a JSON response. The OpenAPI spec acts as the ultimate assertion.
3. **Implicit Coverage:** By adding the filter to your base `RequestSpecification`, every API call used for test data setup in your Hybrid Framework automatically doubles as a contract test.

With our hybrid and contract testing strategies firmly in place, the final step in our API automation journey is **API Framework Design**—architecting a highly reusable backend engine using the Builder Pattern and POJOs. We will cover this in the next article!
