---
title: Introduction to REST APIs & HTTP Methods
date: 2025-02-01
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [restassured, java, automation, api-testing]
category: API Testing
categories: [API Testing, Automation, Java]
excerpt: >-
  Before we write a single line of Java code, we must understand the language of the web. Modern web applications communicate through REST APIs.
readTime: 5 min read
---

Welcome to the **Java RestAssured API Testing Mastery** series! 

Before we write a single line of Java code, we must understand the language of the web. Modern web applications communicate through REST APIs (Representational State Transfer Application Programming Interfaces).

Think of an API like a waiter in a restaurant. You (the client) give the waiter your order (the Request). The waiter takes the order to the kitchen (the Server), and then brings your food (the Response) back to you.

---

## Core HTTP Methods

REST APIs use standard HTTP Methods to tell the server exactly what action we want to perform. Here are the five you will use every day as an Automation Engineer:

**GET** (Read)
Used to retrieve data from the server. For example, fetching a list of users or a specific product details. A `GET` request never alters the database.

**POST** (Create)
Used to send new data to the server. When you submit a signup form, your browser is sending a `POST` request containing your email and password to create a new record in the database.

**PUT** (Update Completely)
Used to completely replace an existing resource. If you want to update your entire user profile, you send a `PUT` request containing all the new details.

**PATCH** (Update Partially)
Used to modify just a single field of an existing resource. If you only want to change your username without touching your email, you use `PATCH`.

**DELETE** (Remove)
Used to permanently remove a resource from the server. 

---

## The Anatomy of an API Request

When we write our RestAssured code, we will be constructing HTTP requests. Every request consists of:
1. **Endpoint/URL:** The exact address we are sending the request to (e.g., `http://localhost:8080/api/users`)
2. **Method:** The action we want to take (GET, POST, etc.)
3. **Headers:** Metadata about the request, such as `Content-Type: application/json` or Authorization tokens.
4. **Body/Payload:** The actual data we are sending (used primarily in POST and PUT requests).

In the next tutorial, we will set up our Java development environment and write our very first RestAssured script to execute these requests automatically!
