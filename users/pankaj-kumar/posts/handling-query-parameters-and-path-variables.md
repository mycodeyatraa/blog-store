---
title: Handling Query Parameters & Path Variables
date: 2025-02-04
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [restassured, java, automation, path-variables, query-parameters]
category: API Testing
categories: [API Testing, Automation, Java]
excerpt: >-
  Real-world API testing requires dynamic URL construction. In this tutorial, we will use RestAssured to inject Query Parameters and Path Variables dynamically.
readTime: 6 min read
---

In our previous tutorial, we successfully sent static GET and POST requests. However, real-world API testing requires dynamic URL construction. You will rarely request "all users"—instead, you will request "page 1 of users" or "a specific user by ID".

In this tutorial, we will use RestAssured to dynamically inject **Query Parameters** and **Path Variables** into our HTTP requests.

---

## 1. Using Query Parameters

Query Parameters are attached to the end of a URL after a question mark ? (e.g., /api/users?page=1&limit=3). They are used to filter, sort, or paginate data.

Instead of hardcoding the URL string, RestAssured provides the .queryParam() method to construct the URL dynamically. This is much safer and easier to maintain.

Let's fetch exactly 3 users from our MyCodeYatra Mock Server:

```
@Test(priority = 1)
public void testQueryParameters() {
    System.out.println("\n--- Executing GET /users with Query Params ---");
    
    // Using queryParam() to build dynamic URLs
    Response response = RestAssured.given()
            .queryParam("page", 1)
            .queryParam("limit", 3)
            .when()
            .get("/users")
            .then()
            .extract().response();

    System.out.println("Status Code: " + response.getStatusCode());
    
    // Extract the list of data items
    List<Object> userList = response.jsonPath().getList("data");
    System.out.println("Number of users returned: " + userList.size());
    
    Assert.assertEquals(response.getStatusCode(), 200);
    Assert.assertEquals(userList.size(), 3, "Pagination limit did not work!");
}
```

## 2. Using Path Variables

Path Variables are part of the URL structure itself (e.g., /api/users/12345). They are typically used to identify a specific resource.

RestAssured uses the .pathParam() method to dynamically inject variables into {placeholder} markers in your URL string. In the following test, we will extract a random user's id and then use it as a Path Variable to fetch their specific profile:

```
@Test(priority = 2)
public void testPathVariables() {
    System.out.println("\n--- Executing GET /users/{id} with Path Variable ---");
    
    // First, fetch the first user to get a valid ID
    Response allUsers = RestAssured.given()
            .queryParam("limit", 1)
            .when()
            .get("/users")
            .then()
            .extract().response();
            
    String targetId = allUsers.jsonPath().getString("data[0].id");
    String targetName = allUsers.jsonPath().getString("data[0].name");

    // Now, use pathParam() to fetch that specific user safely
    Response userResponse = RestAssured.given()
            .pathParam("userId", targetId)
            .when()
            .get("/users/{userId}") // The {userId} marker is replaced dynamically
            .then()
            .extract().response();

    System.out.println("Status Code: " + userResponse.getStatusCode());
    System.out.println("Returned Name: " + userResponse.jsonPath().getString("name"));

    Assert.assertEquals(userResponse.getStatusCode(), 200);
    Assert.assertEquals(userResponse.jsonPath().getString("name"), targetName);
}
```

---

## The Execution Output

When we execute these tests against our Mock Server, here is the exact console output showing flawless execution:

```
--- Executing GET /users with Query Params ---
Status Code: 200
Number of users returned: 3

--- Executing GET /users/{id} with Path Variable ---
Status Code: 200
Returned Name: Taurean Walsh Sr.
[INFO] Tests run: 2, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 18.55 s -- in TestSuite
```

By mastering .queryParam() and .pathParam(), your automation scripts become completely dynamic, capable of adapting to varying test data on every single run!

In our next tutorial, we will explore advanced JSON parsing using Jackson and Gson.
