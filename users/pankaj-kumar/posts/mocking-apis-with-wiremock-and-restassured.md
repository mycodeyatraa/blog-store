---
title: Mocking APIs with WireMock and RestAssured
date: 14-Feb-2025
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [restassured, java, wiremock, mocking, testing]
category: API Testing
categories: [API Testing, Automation, Java]
excerpt: >-
  Eliminate flaky automated tests by simulating third-party HTTP dependencies locally using WireMock's powerful stubbing and verification capabilities.
readTime: 6 min read
---

One of the biggest challenges in API Testing is dealing with **third-party dependencies**. What happens if you need to test how your system handles a payment gateway response, but the actual Stripe or PayPal sandbox environment is temporarily down? 

Your tests fail. This is known as a "flaky test."

To eliminate flaky tests caused by third-party unreliability, we use **Mocking**. And the king of API Mocking in the Java ecosystem is **WireMock**!

---

## 1. What is WireMock?

WireMock is an incredibly powerful simulator for HTTP-based APIs. You start a standalone server directly inside your TestNG `@BeforeClass` setup. This server intercepts HTTP requests and immediately returns hard-coded responses (stubs) that *you* define.

First, add the standalone dependency to your `pom.xml`:

```xml
<dependency>
    <groupId>org.wiremock</groupId>
    <artifactId>wiremock-standalone</artifactId>
    <version>3.6.0</version>
    <scope>test</scope>
</dependency>
```

## 2. Stubbing an API Response

In this example, we will boot up a WireMock server on an isolated port (`8089`). We will configure it so that if *any* GET request hits the `/third-party/payment-gateway` endpoint, it will instantly return a `200 OK` status with a dummy JSON payload containing `{ "status": "SUCCESS" }`.

```java
package com.mycodeyatra.tests;
import com.github.tomakehurst.wiremock.WireMockServer;
import io.restassured.RestAssured;
import org.testng.Assert;
import org.testng.annotations.AfterClass;
import org.testng.annotations.BeforeClass;
import org.testng.annotations.Test;
import static com.github.tomakehurst.wiremock.client.WireMock.*;
public class WireMockTest {
    private WireMockServer wireMockServer;
    @BeforeClass
    public void setup() {
        // Start WireMock on an isolated port
        wireMockServer = new WireMockServer(8089);
        wireMockServer.start();
        // Point RestAssured to the mock server instead of the real internet!
        RestAssured.baseURI = "http://localhost:8089";
        // Stub a mock API endpoint
        wireMockServer.stubFor(get(urlEqualTo("/third-party/payment-gateway"))
                .willReturn(aResponse()
                        .withHeader("Content-Type", "application/json")
                        .withStatus(200)
                        .withBody("{ \\"status\\": \\"SUCCESS\\", \\"transactionId\\": \\"12345ABC\\" }")));
    }
```

## 3. Testing Against the Mock

Now, we write a standard RestAssured test. As far as RestAssured is concerned, it is hitting a real payment gateway on the internet! 

```java
    @Test
    public void testMockedPaymentGateway() {
        System.out.println("\\n--- Executing WireMock Test ---");
        String responseBody = RestAssured.given()
                .when()
                .get("/third-party/payment-gateway")
                .then()
                .statusCode(200)
                .extract().asString();
        System.out.println("Mocked Response Received: " + responseBody);
        Assert.assertTrue(responseBody.contains("12345ABC"));
    }
    @AfterClass
    public void teardown() {
        // Always shut down the mock server after the test suite finishes
        if (wireMockServer != null) {
            wireMockServer.stop();
        }
    }
}
```

## The Execution Output

When you run this test (`mvn clean test`), the execution happens locally without ever reaching out to the internet!

```text
--- Executing WireMock Test ---
Mocked Response Received: { "status": "SUCCESS", "transactionId": "12345ABC" }
[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 14.52 s -- in TestSuite
```

With WireMock, you can simulate timeouts, 500 Internal Server Errors, and incredibly complex JSON structures in milliseconds. It is an absolute must-have for enterprise CI/CD environments where flakiness is unacceptable. 

Speaking of CI/CD, that's exactly what we'll tackle in the next tutorial!
