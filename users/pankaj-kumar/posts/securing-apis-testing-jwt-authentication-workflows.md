---
title: Securing APIs: Testing JWT Authentication Workflows
date: 2025-01-19
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [rest-api, api-testing, automation, security, jwt]
category: API Testing
categories: [API Testing, Automation, Security]
excerpt: >-
  In a real-world production environment, you cannot let anyone access sensitive data. Modern APIs use security mechanisms like JSON Web Tokens (JWT) to ensure only authorized users can perform actions.
readTime: 4 min read
---

Up until this point, all of the endpoints we have interacted with on our mock server have been completely open. In a real-world production environment, you cannot simply let anyone access, modify, or delete sensitive data!

Modern APIs use security mechanisms to ensure that only authorized users can perform certain actions. The most widely adopted industry standard for this is **JSON Web Tokens (JWT)**.

In this tutorial, we will explore how to test secure endpoints by generating and passing JWT authentication tokens to the MyCodeYatra Mock API Server.

> **Important Resource:** The entire codebase for this tutorial is hosted on our official GitHub Repository. You can find it here: [https://github.com/MYCodeYatra/myct-api-test-server](https://github.com/MYCodeYatra/myct-api-test-server)

---

## What is a JSON Web Token (JWT)?

Think of a JWT as a digital wristband at a music festival. 

When you first arrive, security checks your ticket (your username and password) and hands you a wristband (the token). For the rest of the day, instead of showing your ticket at every single tent, you simply flash your wristband. 

In API terms, you send a single login request to generate a token, and then you attach that token to all subsequent requests to prove your identity.

---

## Generating an Authentication Token

Before we can access any secure endpoints, we need our wristband. We do this by sending a POST request to our server's login endpoint.

Open your terminal and run the following command to submit dummy login credentials:

```bash
curl -X POST http://localhost:8080/api/auth/login \
-H "Content-Type: application/json" \
-d '{"email": "admin@example.com", "password": "password123"}'
```

If the credentials are valid, the server will respond with a `200 OK` status and a JSON body containing your shiny new access token.

```json
{
  "status": "success",
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6MTIz...",
  "user": {
    "role": "admin",
    "email": "admin@example.com"
  }
}
```

Copy that long `token` string. You will need it for the next step!

---

## Accessing a Secure Endpoint

Our mock server contains a special, hidden endpoint at `/api/secure-data` that is strictly protected. If you try to perform a GET request on it without a token, the server will block you and return a `401 Unauthorized` error.

To successfully access it, we must include our JWT in the **Headers** of our request. Specifically, we use the `Authorization` header and prefix our token with the word `Bearer`.

Run this command in your terminal, making sure to replace the placeholder with the actual token you generated in the previous step:

```bash
curl -X GET http://localhost:8080/api/secure-data \
-H "Authorization: Bearer YOUR_LONG_TOKEN_STRING"
```

Because your request now contains a valid security wristband, the server will successfully validate it and return a `200 OK` response with the highly classified secure data!

### Wrapping Up

Authentication is the absolute backbone of API security. By mastering the login-to-token workflow, you can script robust automated test suites that simulate a user logging in, capturing their dynamic token, and reusing it across dozens of subsequent secure requests.

In the **next blog**, we will push our server to the limit. We will learn how to test an API's resilience by simulating extreme latency, forcing network timeouts, and intentionally injecting server errors!
