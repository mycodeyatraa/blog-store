---
title: Automating GraphQL Queries and Mutations in Postman
date: 2025-01-29
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [postman, graphql, automation]
category: API Testing
categories: [API Testing, Automation, Postman]
excerpt: >-
  GraphQL solves over-fetching by allowing explicit queries. In this tutorial, we will execute GraphQL Queries and Mutations natively within Postman.
readTime: 5 min read
---

While REST APIs have been the industry standard for a decade, **GraphQL** is rapidly taking over. GraphQL solves the massive problem of "over-fetching" data by allowing the client to explicitly ask for only the fields it needs.

Because every GraphQL request is essentially just an HTTP POST containing a JSON payload, Postman is fully equipped to handle it! In this tutorial, we will execute GraphQL Queries and Mutations against our local MyCodeYatra Mock API Server.

---

## Executing a GraphQL Query

Make sure your local mock server is running. In GraphQL, there is only one endpoint. 

Open Postman, create a new standard HTTP Request, and set the method to **POST**. Set the URL to:

`http://localhost:8080/graphql`

Now, go to the **Body** tab. Instead of selecting `raw` and `JSON`, select the dedicated **GraphQL** option.

In the Query box, type the following payload to fetch only the names of our users:

```graphql
query { 
  users { 
    data { 
      name 
    } 
  } 
}
```

Hit **Send**. Postman will immediately return a JSON response containing exactly the data you asked for, cleanly omitting emails, roles, and timestamps!

---

## Using GraphQL Variables

Hardcoding values into your GraphQL queries is dangerous and inefficient. Postman supports passing dedicated GraphQL variables.

In the same request, update your query box to look like this:

```graphql
query GetUserById($targetId: ID!) { 
  user(id: $targetId) { 
    name 
    email 
  } 
}
```

Below the Query box, open the **GraphQL Variables** panel and provide the JSON variable payload:

```json
{
  "targetId": "a1b2c3d4-e5f6-7g8h-9i0j-k1l2m3n4o5p6"
}
```

Hit **Send**. Postman cleanly separates the query structure from the data, successfully fetching the specific user!

---

## Writing a GraphQL Mutation

Queries are for reading data. To modify data, GraphQL uses **Mutations**. Let's create a new user.

Update your Postman query box to the following:

```graphql
mutation CreateNewUser($name: String!, $email: String!, $role: String!) { 
  createUser(name: $name, email: $email, role: $role) { 
    id 
    name 
  } 
}
```

Update your GraphQL Variables panel to provide the required fields:

```json
{
  "name": "Postman Master",
  "email": "postman@example.com",
  "role": "admin"
}
```

When you hit **Send**, the backend server will create the new user and instantly echo back the newly generated `id` alongside the name.

### Wrapping Up

Postman's native GraphQL support makes it incredibly easy to format, test, and save complex queries and mutations. By mastering these advanced protocols, your API testing skills remain cutting-edge.

In our **final blog**, we will bring everything together in a massive end-to-end Capstone Project!
