---
title: Testing GraphQL APIs with RestAssured
date: 09-Feb-2025
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [restassured, java, graphql, api-testing, automation]
category: API Testing
categories: [API Testing, Automation, Java]
excerpt: >-
  Demystify GraphQL testing by learning how to automate Queries and Mutations seamlessly using RestAssured's powerful JSON processing capabilities.
readTime: 6 min read
---

While REST APIs dominate the web, **GraphQL** has seen massive adoption over the last few years because it allows clients to specifically request exactly the data they need and nothing more.

But did you know that underneath the hood, a GraphQL request is almost always just a standard HTTP POST request with a JSON body?

Because of this, testing GraphQL with RestAssured is surprisingly simple! In this tutorial, we will automate both GraphQL **Queries** (fetching data) and **Mutations** (creating data).

---

## 1. GraphQL Queries (Fetching Data)

In a REST API, you hit multiple different endpoints (e.g., `/users`, `/posts`). In GraphQL, there is typically only **one** endpoint (e.g., `/graphql`).

To fetch data, you send a POST request with a JSON object containing a `query` string.

Let's test our MyCodeYatra Mock API's GraphQL endpoint to fetch a list of users:

```java
@Test
public void testGraphQLQuery() {
    System.out.println("\\n--- Executing GraphQL Query Test ---");
    // GraphQL Query Payload
    String graphqlPayload = "{\\n" +
            "  \\"query\\": \\"{ users { id name email role } }\\"\\n" +
            "}";
    // Execute POST Request
    Response response = RestAssured.given()
            .contentType(ContentType.JSON)
            .body(graphqlPayload)
            .when()
            .post("/graphql")
            .then()
            .extract().response();
    // Validate Status Code
    Assert.assertEquals(response.getStatusCode(), 200);
    // Assert data exists inside the 'data' node
    Assert.assertNotNull(response.jsonPath().getList("data.users"));
    Assert.assertFalse(response.jsonPath().getList("data.users").isEmpty());
}
```

Notice that the response payload encapsulates everything inside a `"data"` root node!

## 2. GraphQL Mutations (Modifying Data)

When you want to create or update data, GraphQL uses a `mutation`. It works exactly the same way as a Query—it's just a POST request where the string starts with `mutation`.

Let's use a `Map` payload structure this time, which is cleaner than raw string concatenation:

```java
@Test
public void testGraphQLMutation() {
    System.out.println("\\n--- Executing GraphQL Mutation Test ---");
    // GraphQL Mutation Payload
    Map<String, Object> mutationPayload = new HashMap<>();
    mutationPayload.put("query", "mutation { createUser(name: \\"GraphQL User\\", email: \\"graphql@example.com\\", role: \\"admin\\") { id name email } }");
    Response response = RestAssured.given()
            .contentType(ContentType.JSON)
            .body(mutationPayload)
            .when()
            .post("/graphql")
            .then()
            .extract().response();
    // Validate Status Code
    Assert.assertEquals(response.getStatusCode(), 200);
    // Validate the mutated user object returned
    String createdName = response.jsonPath().getString("data.createUser.name");
    System.out.println("Created User Name: " + createdName);
    Assert.assertEquals(createdName, "GraphQL User");
}
```

---

## The Execution Output

When we execute our TestNG suite (`mvn clean test`), we can see the GraphQL automation succeeding:

```text
--- Executing GraphQL Mutation Test ---
Created User Name: GraphQL User
--- Executing GraphQL Query Test ---
GraphQL Response: 
{
    "data": {
        "users": [
            {
                "id": "834b9d...",
                "name": "Jane Doe",
                "email": "jane@example.com",
                "role": "user"
            }
        ]
    }
}
[INFO] Tests run: 2, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 14.52 s -- in TestSuite
```

By leveraging RestAssured's JSON capabilities, you can build powerful regression suites that assert your GraphQL API's exact graphical payload requirements.

In our final tutorial of this series, we will tackle WebSockets and wrap up the entire REST Assured testing framework!
