---
title: Advanced GraphQL: Using Variables and Dynamic Filters
date: 2025-01-22
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [graphql, api-testing, automation, variables, dynamic-queries]
category: API Testing
categories: [API Testing, Automation, GraphQL]
excerpt: >-
  Real applications don't just fetch static data. They fetch dynamic data based on user input. In this tutorial, we will learn the correct way to pass dynamic data into our queries using GraphQL Variables.
readTime: 4 min read
---

In the previous tutorial, we learned how to write a basic GraphQL query to fetch exactly the fields we wanted from our mock server. However, real applications don't just fetch static data. They need to fetch dynamic data based on user input—like searching for a specific user ID or filtering by a role.

In REST, we accomplish this by concatenating variables into the URL path or using query parameters. In GraphQL, doing that is considered a dangerous anti-pattern!

In this tutorial, we will learn the correct way to pass dynamic data into our queries using **GraphQL Variables**.

> **Important Resource:** The entire codebase for this tutorial is hosted on our official GitHub Repository. You can find it here: [https://github.com/MYCodeYatra/myct-api-test-server](https://github.com/MYCodeYatra/myct-api-test-server)

---

## The Problem with String Interpolation

If you are writing an automated test to fetch a user with the ID `123`, you might be tempted to just inject that ID directly into your GraphQL query string, like this:

`query { user(id: "123") { name } }`

This is bad practice for two reasons. First, it forces the GraphQL server to recompile the query from scratch every single time the ID changes, destroying server performance. Second, if that ID comes from an external source, you are opening your application up to dangerous injection attacks.

---

## Defining GraphQL Variables

The correct approach is to separate the *structure* of your query from the *data* it uses. 

First, we define a variable name inside our query string. In GraphQL, variables always start with a dollar sign `$`. We must also explicitly define the variable's type.

Take a look at the structure of this query:

`query GetUserById($targetId: ID!) { user(id: $targetId) { name email } }`

Here, we declare a variable called `$targetId` and tell the engine that it is a required `ID` type (the exclamation mark `!` means it is mandatory). We then pass that variable into the `user` lookup.

---

## Executing a Dynamic Query

To actually execute this dynamic query against the MyCodeYatra Mock API Server, we send our JSON POST request, but this time we provide a completely separate `variables` object alongside our `query` string.

Let's execute this against the local mock server! Open your terminal and run the following command (make sure you use an ID that actually exists in your mock database, or the server will return null):

```bash
curl -X POST http://localhost:8080/graphql \
-H "Content-Type: application/json" \
-d '{"query": "query GetUserById($targetId: ID!) { user(id: $targetId) { name email role } }", "variables": { "targetId": "a1b2c3d4-e5f6-7g8h-9i0j-k1l2m3n4o5p6" }}'
```

The server cleanly separates the logic, injects your variable safely, and returns the requested user:

```json
{
  "data": {
    "user": {
      "name": "Test User",
      "email": "test@example.com",
      "role": "user"
    }
  }
}
```

---

## Complex Filtering with Variables

Variables aren't just for single lookups. You can use them to dynamically paginate or filter massive lists of data.

Let's fetch a list of users, but use variables to strictly define how many users we want, and to filter only for users who have the "admin" role.

Run this advanced command:

```bash
curl -X POST http://localhost:8080/graphql \
-H "Content-Type: application/json" \
-d '{"query": "query GetAdmins($limit: Int, $roleFilter: String) { users(limit: $limit, role: $roleFilter) { data { name role } } }", "variables": { "limit": 2, "roleFilter": "admin" }}'
```

The GraphQL engine will execute the query, apply the filter variable, limit the results to exactly 2 records, and return the filtered JSON payload!

### Wrapping Up

By decoupling your query logic from your dynamic data using GraphQL Variables, you write automation scripts that are drastically more secure, reusable, and performant. 

In the **next blog**, we will explore how to modify data in the GraphQL ecosystem. We will leave `query` behind and learn how to write our first GraphQL **Mutation**!
