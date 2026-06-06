---
title: Beyond REST: An Introduction to GraphQL Querying
date: 2025-01-21
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [graphql, api-testing, automation, rest-api, queries]
category: API Testing
categories: [API Testing, Automation, GraphQL]
excerpt: >-
  REST is fantastic, but it has a massive flaw: over-fetching data. Enter GraphQL, a modern query language that allows clients to ask the server for exactly what they need, and nothing more.
readTime: 4 min read
---

For the last seven tutorials, we have been working exclusively with REST endpoints. REST is fantastic, but it has a massive flaw: over-fetching.

When we hit the `/api/users` REST endpoint, the server returns the entire user object—including the ID, name, email, role, and creation timestamp. But what if our frontend application only needs to display a list of names? The server still forces us to download the emails, roles, and timestamps, wasting precious mobile data and bandwidth.

Enter **GraphQL**. Developed by Facebook, GraphQL is a query language that allows clients to ask the server for exactly what they need, and nothing more.

In this post, we will execute our very first GraphQL query against the MyCodeYatra Mock API Server!

> **Important Resource:** The entire codebase for this tutorial is hosted on our official GitHub Repository. You can find it here: [https://github.com/MYCodeYatra/myct-api-test-server](https://github.com/MYCodeYatra/myct-api-test-server)

---

## The Single Endpoint Architecture

In REST, you interact with multiple endpoints (like `/api/users`, `/api/posts`, `/api/auth`). 

In GraphQL, there is only ONE endpoint. Every single request you make—whether you are reading data or modifying it—is sent as a POST request to this single unified address.

For our mock server, that address is `http://localhost:8080/graphql`.

---

## Writing Your First Query

Unlike REST, where the structure of the returned data is defined strictly by the backend developer, GraphQL allows the frontend to dictate the structure.

We do this by sending a JSON payload containing a special `query` string. This string tells the GraphQL engine exactly which fields we want to extract from the database.

Let's test this! We want to fetch all the users from the database, but we ONLY want their names. We do not want their IDs, their emails, or their roles.

Open your terminal and execute this highly targeted query:

```bash
curl -X POST http://localhost:8080/graphql \
-H "Content-Type: application/json" \
-d '{"query": "query { users { data { name } } }"}'
```

The mock server will instantly process the query and return a beautifully slim JSON object. Look closely at the response:

```json
{
  "data": {
    "users": {
      "data": [
        { "name": "Alice Johnson" },
        { "name": "Bob Smith" }
      ]
    }
  }
}
```

Notice how clean that is! The emails and timestamps are completely gone. We asked the server exclusively for the `name` field, and the server respected our request.

---

## Expanding the Query

What if we realize we actually do need the emails for a different screen in our app? 

With REST, we would have to ask the backend team to build a completely new endpoint or add a new query parameter. With GraphQL, we simply modify our query string to include the `email` field alongside the `name` field.

Run this expanded query in your terminal:

```bash
curl -X POST http://localhost:8080/graphql \
-H "Content-Type: application/json" \
-d '{"query": "query { users { data { name email } } }"}'
```

Instantly, without the backend server code changing at all, the response updates to include the newly requested data!

```json
{
  "data": {
    "users": {
      "data": [
        { "name": "Alice Johnson", "email": "alice.j@example.com" },
        { "name": "Bob Smith", "email": "bob.smith@example.com" }
      ]
    }
  }
}
```

### Wrapping Up

GraphQL represents a paradigm shift in how applications fetch data. By empowering the client to ask for exactly what it needs, GraphQL drastically reduces network overhead and speeds up frontend performance.

In the **next blog**, we will explore how to perform advanced GraphQL operations by dynamically filtering our queries using GraphQL variables!
