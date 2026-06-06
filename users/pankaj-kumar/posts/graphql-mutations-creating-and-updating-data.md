---
title: GraphQL Mutations: Creating and Updating Data
date: 2025-01-23
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [graphql, api-testing, automation, mutations, crud]
category: API Testing
categories: [API Testing, Automation, GraphQL]
excerpt: >-
  In REST, we use specific HTTP methods like POST and PUT to modify data. In GraphQL, everything goes through a single endpoint. To modify data, GraphQL introduces the powerful concept of a Mutation.
readTime: 4 min read
---

In the REST architecture, we use specific HTTP methods like POST, PUT, and DELETE to modify data. In GraphQL, things operate entirely differently. Because every single request in GraphQL is sent as an HTTP POST request to a single endpoint, we cannot rely on HTTP verbs to tell the server what we want to do.

Instead, GraphQL introduces the concept of a **Mutation**. While a "Query" is strictly used for reading data, a "Mutation" is strictly used for creating, updating, or deleting data.

In this tutorial, we will execute our very first GraphQL Mutation against the MyCodeYatra Mock API Server!

> **Important Resource:** The entire codebase for this tutorial is hosted on our official GitHub Repository. You can find it here: [https://github.com/MYCodeYatra/myct-api-test-server](https://github.com/MYCodeYatra/myct-api-test-server)

---

## Writing a Create Mutation

Let's simulate a user registration flow. We want to create a brand new user in the database. 

To do this, we write a Mutation string. Just like we learned in the previous blog, we will use GraphQL Variables to safely pass in the new user's data (name, email, and role).

Notice how similar the structure is to a Query. The only difference is that we start the string with the word `mutation` instead of `query`. Furthermore, GraphQL is brilliant—it allows us to immediately query the data we just created in the exact same request! We will ask the server to create the user, and then immediately return the newly generated `id` and `createdAt` timestamp.

Open your terminal and run this complete command:

```bash
curl -X POST http://localhost:8080/graphql \
-H "Content-Type: application/json" \
-d '{
  "query": "mutation CreateNewUser($name: String!, $email: String!, $role: String!) { createUser(name: $name, email: $email, role: $role) { id name email role createdAt } }",
  "variables": { "name": "GraphQL Master", "email": "graphql@example.com", "role": "admin" }
}'
```

The server instantly processes the payload, creates the record in the in-memory database, and echoes back the exact fields we requested!

```json
{
  "data": {
    "createUser": {
      "id": "e4f8d2b2-6c8a-493b-b2f5-8d9e2b1a8c90",
      "name": "GraphQL Master",
      "email": "graphql@example.com",
      "role": "admin",
      "createdAt": "2025-01-23T14:22:11.000Z"
    }
  }
}
```

---

## Updating Data with Mutations

Updating a record uses the exact same `mutation` syntax. The only difference is the specific resolver function we call inside the string. Instead of `createUser`, we will call `updateUser`.

When updating, we must provide the unique `id` of the record we want to modify, alongside the new data fields. Let's update the user we just created to have a standard user role instead of an admin role.

Copy the `id` from the previous response, and execute this update mutation in your terminal:

```bash
curl -X POST http://localhost:8080/graphql \
-H "Content-Type: application/json" \
-d '{
  "query": "mutation UpdateExistingUser($id: ID!, $role: String!) { updateUser(id: $id, role: $role) { id name role } }",
  "variables": { "id": "e4f8d2b2-6c8a-493b-b2f5-8d9e2b1a8c90", "role": "user" }
}'
```

The server updates the record and returns our explicitly requested fields, proving the role has been successfully downgraded!

---

## Deleting Data

Deleting data is the simplest mutation of all. We simply call the `deleteUser` resolver and pass the target `id`.

Run this final command to destroy our test record:

```bash
curl -X POST http://localhost:8080/graphql \
-H "Content-Type: application/json" \
-d '{
  "query": "mutation DeleteAUser($id: ID!) { deleteUser(id: $id) }",
  "variables": { "id": "e4f8d2b2-6c8a-493b-b2f5-8d9e2b1a8c90" }
}'
```

The server executes the deletion and returns a simple boolean `true` to confirm the record no longer exists.

### Wrapping Up

You have now mastered the complete CRUD lifecycle in GraphQL! By leveraging Mutations alongside secure GraphQL Variables, you can script incredibly powerful, dynamic automation tests that interact with modern backend architectures.

In the **next blog**, we will push past simple HTTP requests. We will learn how to establish persistent, real-time connections by testing **WebSockets**!
