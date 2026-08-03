---
title: Handling Authentication: OAuth 2.0, Bearer Tokens & JWT in REST-Assured
date: 18-Feb-2026
lastUpdated: 18-Feb-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: ["restassured", "java", "oauth2", "jwt", "bearer-token", "security"]
category: REST-Assured
categories: ["REST-Assured", "API Testing", "Java"]
excerpt: >-
  Secure your API automation! Learn how to handle OAuth 2.0 client credentials, Bearer tokens, and JWT authentication in REST-Assured.
readTime: 8 min read
---

# Handling Authentication: OAuth 2.0, Bearer Tokens & JWT in REST-Assured

Enterprise microservices protect access using modern authentication schemes such as **OAuth 2.0**, **JSON Web Tokens (JWT)**, and **Bearer Tokens**.

Automating secure API workflows requires fetching authorization tokens dynamically during setup and passing them inside HTTP headers for downstream requests.

---

## 1. Authentication Mechanisms Overview

- **Basic Auth**: Base64 encoded `username:password` passed in `Authorization` header.
- **Bearer / JWT Token**: Signed cryptographic token passed as `Authorization: Bearer <token>`.
- **OAuth 2.0**: Protocol where client exchanges `client_id` and `client_secret` for an access token via an Authorization Server.

---

## 2. Implementing Dynamic Token Retrieval & Authentication

Below is a complete test suite demonstrating OAuth 2.0 token acquisition and Bearer token API requests:

**src/test/java/com/mycodeyatra/AuthTestingTest.java**

```java
package com.mycodeyatra;
 
import io.restassured.RestAssured;
import io.restassured.http.ContentType;
import org.junit.jupiter.api.BeforeAll;
import org.junit.jupiter.api.Test;
 
import static io.restassured.RestAssured.given;
import static org.hamcrest.Matchers.*;
 
public class AuthTestingTest {
 
    private static String accessToken;
 
    @BeforeAll
    public static void obtainOAuth2Token() {
        RestAssured.baseURI = "https://mycodeyatra.com/api";
 
        // Fetch Access Token from Auth Server
        accessToken = given()
            .contentType(ContentType.URLENC)
            .formParam("grant_type", "client_credentials")
            .formParam("client_id", "my_automation_client_id")
            .formParam("client_secret", "my_automation_secret_key")
        .when()
            .post("/oauth/token")
        .then()
            .statusCode(200)
            .extract()
            .path("access_token");
    }
 
    @Test
    public void testProtectedEndpointWithBearerToken() {
        given()
            .header("Authorization", "Bearer " + accessToken)
            .contentType(ContentType.JSON)
        .when()
            .get("/v1/secure-dashboard")
        .then()
            .statusCode(200)
            .body("status", equalTo("AUTHORIZED"))
            .body("userRole", equalTo("AUTOMATION_ADMIN"));
    }
 
    @Test
    public void testBuiltInOAuth2Method() {
        given()
            .auth().oauth2(accessToken) // REST-Assured built-in OAuth2 shortcut
            .contentType(ContentType.JSON)
        .when()
            .get("/v1/user-settings")
        .then()
            .statusCode(200);
    }
}
```

---

## Security Best Practices

1. **Never Hardcode Secrets**: Store `client_secret` and API credentials in environment variables or CI/CD secret vaults (e.g., GitHub Secrets, AWS Secrets Manager).
2. **Token Caching**: Cache tokens during test suite execution to prevent hitting token rate limits.
