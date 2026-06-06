---
title: Parsing and Validating XML Responses
date: 07-Feb-2025
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [restassured, java, xml, xpath, automation]
category: API Testing
categories: [API Testing, Automation, Java]
excerpt: >-
  Learn how to effortlessly parse, navigate, and validate XML payloads in legacy or enterprise APIs using RestAssured's built-in xmlPath() tools.
readTime: 5 min read
---

While JSON is the undisputed king of modern REST APIs, many legacy enterprise systems—particularly SOAP services or older banking backends—still communicate exclusively in **XML**.

Fortunately, RestAssured treats XML just as beautifully as it treats JSON! In this tutorial, we will learn how to extract data and validate XML structures using `xmlPath()`.

---

## 1. Fetching an XML Response

First, we need an endpoint that returns XML. Our MyCodeYatra Mock API Server has a special "chaos" endpoint at `/api/chaos/xml` designed exactly for this purpose. 

When you hit this endpoint, it returns the following raw XML payload:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<users>
  <user>
    <id>1</id>
    <name>Alice</name>
  </user>
</users>
```

Let's write a standard RestAssured GET request to fetch this data and ensure the status code is `200 OK`.

```java
@Test
public void testXmlParsing() {
    System.out.println("\\n--- Executing XML Parsing Test ---");
    Response response = RestAssured.given()
            .when()
            .get("/chaos/xml")
            .then()
            .extract().response();
    // Validate the status code
    Assert.assertEquals(response.getStatusCode(), 200);
    System.out.println("Raw XML Response:");
    System.out.println(response.getBody().asPrettyString());
}
```

## 2. Navigating the XML Tree

When working with JSON, we used `jsonPath()`. For XML, RestAssured provides `.xmlPath()`. 

Unlike JSON, which relies on brackets and keys, XML is hierarchical. We navigate through the tree using **dot notation**, tracing the path from the root node (`<users>`) down to the target element (`<name>`).

If multiple identical nodes exist (like an array of `<user>` elements), we can access them using standard `[0]` index notation!

Let's extract Alice's ID and Name directly from the payload:

```java
// Extracting data using xmlPath()
String firstUserName = response.xmlPath().getString("users.user[0].name");
int firstUserId = response.xmlPath().getInt("users.user[0].id");
System.out.println("Extracted User ID: " + firstUserId);
System.out.println("Extracted User Name: " + firstUserName);
// Assert on the extracted values
Assert.assertEquals(firstUserName, "Alice");
Assert.assertEquals(firstUserId, 1);
```

---

## The Execution Output

When we run this test via our TestNG suite (`mvn clean test`), here is the exact console output showing successful XML extraction:

```text
--- Executing XML Parsing Test ---
Raw XML Response:
<?xml version="1.0" encoding="UTF-8"?>
<users>
  <user>
    <id>1</id>
    <name>Alice</name>
  </user>
</users>
Extracted User ID: 1
Extracted User Name: Alice
[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 13.52 s -- in TestSuite
```

As you can see, extracting data from an XML body is just as intuitive as parsing JSON. 

In the next tutorial, we will dive into API File Uploads and Downloads using RestAssured!
