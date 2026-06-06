---
title: Mastering Pagination and Sorting in REST API Tests
date: 18-Jan-2025
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [rest-api, api-testing, automation, pagination, sorting]
category: API Testing
categories: [API Testing, Automation, REST API]
excerpt: >-
  In real-world applications, databases hold millions of records. To prevent servers from crashing, APIs return data in small, manageable chunks called pages. In this tutorial, we will learn how to master pagination and sorting.
readTime: 4 min read
---

In real-world applications, databases hold millions of records. If a frontend application or a mobile app requested all of those records at once, the server would crash and the app would freeze.

To prevent this, APIs return data in small, manageable chunks called "pages". As an automation engineer, you must know how to test that an API correctly slices and orders this data.

In this tutorial, we will learn how to use the MyCodeYatra Mock API Server to master Pagination and Sorting!

> **Important Resource:** The entire codebase for this tutorial is hosted on our official GitHub Repository. You can find it here: [https://github.com/MYCodeYatra/myct-api-test-server](https://github.com/MYCodeYatra/myct-api-test-server)

---

## The Concept of Pagination

When you perform a standard GET request to our `/api/users` endpoint, you might notice that the server does not return all 50 generated users at once. Instead, it returns a JSON object containing pagination metadata.

This metadata tells you exactly where you are in the dataset. It includes the current page number, the limit (how many records are allowed per page), the total number of records, and the actual array of data.

```json
{
  "page": 1,
  "limit": 10,
  "total": 50,
  "data": [ ... ]
}
```

By default, our mock server assumes you want `page=1` and `limit=10`. But we can change this behavior dynamically!

---

## Traversing Pages

To navigate through the data, we use URL Query Parameters. We can explicitly tell the server which page we want to view, and how large that page should be.

Let's fetch the second page of data, and let's tell the server we only want 5 records per page instead of the default 10.

Open your terminal and run this customized GET request:

```bash
curl -X GET "http://localhost:8080/api/users?page=2&limit=5"
```

The server will instantly respond with updated metadata reflecting your request. The `data` array will now contain exactly 5 records, representing users 6 through 10 from the database!

```json
{
  "page": 2,
  "limit": 5,
  "total": 50,
  "data": [ ... ]
}
```

In your automation scripts, you can write assertions to verify that the length of the `data` array strictly matches the `limit` query parameter you passed in the URL.

---

## Dynamic Sorting

Retrieving chunked data is great, but users usually want to see the newest data first. This is where sorting comes in.

Our mock server allows you to sort the returned data based on any field in the database. You use the `sort` query parameter to define the field. If you want the sorting to be descending (newest first), you simply prefix the field name with a minus sign `-`.

Let's fetch the first page of users, but this time, let's sort them by their creation date in descending order so the newest users appear at the top.

Execute the following command in your terminal:

```bash
curl -X GET "http://localhost:8080/api/users?sort=-createdAt"
```

Look closely at the `createdAt` timestamps in the returned JSON. You will notice they are perfectly ordered from newest to oldest.

You can also combine sorting with pagination! To fetch the third page of users, with 15 users per page, sorted alphabetically by their name, you would run:

```bash
curl -X GET "http://localhost:8080/api/users?page=3&limit=15&sort=name"
```

### Wrapping Up

Pagination and sorting are two of the most critical functionalities in modern APIs. By chaining query parameters like `?page=2&limit=5&sort=-createdAt`, you can rigorously test exactly how the backend engine slices and organizes database records.

In the **next blog**, we will step into the world of Security. We will leave standard public endpoints behind and learn how to generate and validate JWT Authentication tokens!
