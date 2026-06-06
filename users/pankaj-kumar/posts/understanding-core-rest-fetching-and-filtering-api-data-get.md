---
title: Understanding Core REST: Fetching and Filtering API Data (GET)
date: 16-Jan-2025
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [rest-api, api-testing, automation, mock-server, http-get]
category: API Testing
categories: [API Testing, Automation, REST API]
excerpt: >-
  At the heart of every API test automation framework lies the GET request. In this tutorial, we will explore the core REST endpoints provided by the Mock API Server and extract data using query parameters.
readTime: 4 min read
---

Now that our mock server is successfully running (either via Node.js or Docker), it is time to start interacting with it!

At the heart of every API test automation framework lies the **GET request**. This is the fundamental HTTP method used to retrieve data from a backend server. In this tutorial, we will explore the core REST endpoints provided by the MyCodeYatra Mock API Server and learn how to extract exactly what we need using query parameters.

> **Important Resource:** The entire codebase for this tutorial is hosted on our official GitHub Repository. You can find it here: [https://github.com/MYCodeYatra/myct-api-test-server](https://github.com/MYCodeYatra/myct-api-test-server)

---

## The Base URL and Data Seed

Before we fire any requests, it is important to understand how our server handles data. The MyCodeYatra Mock API Server uses an in-memory database.

Every single time the server boots up, it dynamically seeds the database with 50 unique, completely fake user profiles. This means you will always have fresh data to test against, and any modifications you make during testing will disappear the next time you restart the server.

The server runs on your local machine, so the Base URL for all of our requests will be `http://localhost:8080`.

---

## Fetching All Users

Let's make our very first API call. We want to retrieve the entire list of users currently stored in the database.

Open Postman, or use your terminal with `curl`, and make a simple **GET** request to the `/api/users` endpoint.

Here is the exact command you can run in your terminal:

```bash
curl -X GET http://localhost:8080/api/users
```

The server will respond with a `200 OK` status code and a massive JSON object. The response is wrapped in a pagination object (which we will cover deeply in a future blog), but if you look at the `data` array, you will see all of the dynamically generated users.

```json
{
  "page": 1,
  "limit": 10,
  "total": 50,
  "data": [
    {
      "id": "e4f8d2b2-6c8a-493b-b2f5-8d9e2b1a8c90",
      "name": "Alice Johnson",
      "email": "alice.j@example.com",
      "role": "admin",
      "createdAt": "2025-01-10T14:22:11.000Z"
    },
    {
      "id": "b3e2d1c4-5b7a-482a-a1e4-7c8d1b2a9f81",
      "name": "Bob Smith",
      "email": "bob.smith@example.com",
      "role": "user",
      "createdAt": "2025-01-12T09:15:43.000Z"
    }
  ]
}
```

---

## Fetching a Specific User

Sometimes in automation testing, you do not want all 50 users. You only want to validate the data profile of one specific user.

To do this, we use path variables. Look at the JSON response above and grab one of the `id` values (for example, `b3e2d1c4-5b7a-482a-a1e4-7c8d1b2a9f81`).

Append that specific ID to the end of your URL path, right after `/api/users/`. 

Run the following command:

```bash
curl -X GET http://localhost:8080/api/users/b3e2d1c4-5b7a-482a-a1e4-7c8d1b2a9f81
```

The server will return a `200 OK` response containing ONLY the JSON object for Bob Smith! If you pass an ID that does not exist in the database, the server will correctly return a `404 Not Found` error. This is a perfect negative test scenario for your automation scripts!

---

## Filtering Users by Attributes

What if we want to fetch multiple users, but only those who fit a specific criteria? For example, what if we only want to fetch users who possess the "admin" role?

REST APIs handle this using **Query Parameters**. These are appended to the very end of the URL after a question mark `?`.

To filter the database for administrators, we simply pass `role=admin` into the URL.

Run this filtered request in your terminal:

```bash
curl -X GET "http://localhost:8080/api/users?role=admin"
```

The response will now only contain users where the role is explicitly set to admin! You can use this query parameter logic in your automation frameworks to validate that backend search filters are functioning correctly.

### Wrapping Up

We have successfully mastered the GET method! We fetched a massive array of records, isolated a single record using path variables, and filtered records dynamically using query parameters. 

In the **next blog**, we will get destructive. We will move beyond just reading data and learn how to actively modify the database using the POST, PUT, and DELETE methods!
