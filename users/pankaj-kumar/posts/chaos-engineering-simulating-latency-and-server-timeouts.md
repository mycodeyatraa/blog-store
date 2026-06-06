---
title: Chaos Engineering: Simulating Latency and Server Timeouts
date: 20-Jan-2025
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [rest-api, api-testing, automation, chaos-engineering, reliability]
category: API Testing
categories: [API Testing, Automation, Reliability]
excerpt: >-
  In the real world, servers get overloaded and network connections drop. In this tutorial, we will perform Chaos Engineering by intentionally sabotaging our own API requests to simulate severe latency.
readTime: 4 min read
---

When testing applications, engineers often make the critical mistake of only testing the "Happy Path". They assume that the backend API will always return data instantly and flawlessly. 

In the real world, servers get overloaded, network connections drop, and databases lock up. If your frontend application does not handle these failures gracefully, your users will stare at an infinite loading spinner.

In this tutorial, we will use the MyCodeYatra Mock API Server to perform **Chaos Engineering**. We will intentionally sabotage our own API requests to simulate severe network latency and server timeouts.

> **Important Resource:** The entire codebase for this tutorial is hosted on our official GitHub Repository. You can find it here: [https://github.com/MYCodeYatra/myct-api-test-server](https://github.com/MYCodeYatra/myct-api-test-server)

---

## Simulating Network Latency

Normally, when you run our local mock server, requests complete in less than 50 milliseconds. But what happens if your user is on a slow 3G cellular network? Does your application's loading animation work correctly?

We can force the server to arbitrarily delay its response by using the `delay` query parameter. You pass the desired delay in milliseconds.

Let's tell the server to fetch our users list, but completely pause execution for a massive 5000 milliseconds (5 seconds) before responding. Run this command in your terminal:

```bash
curl -X GET "http://localhost:8080/api/users?delay=5000"
```

When you hit enter, your terminal will freeze. Five seconds later, the JSON data will finally stream onto your screen! 

By injecting this delay parameter into your automated UI tests, you can reliably trigger and validate frontend loading states without needing clumsy third-party network throttling tools.

---

## Forcing Server Errors

Latency is one thing, but complete failure is another. How does your automation framework handle a backend crash? Does it retry the request? Does it log the error gracefully, or does the entire test suite explode?

Our mock server provides a dedicated chaos endpoint designed specifically to fail. The `/api/chaos/error` endpoint allows you to forcibly inject standard HTTP error codes into the response.

Let's simulate a scenario where the database connection has dropped, triggering an internal server crash. We will inject a `500 Internal Server Error` status code.

Run the following chaos command:

```bash
curl -X GET "http://localhost:8080/api/chaos/error?status=500"
```

Instead of data, the server will instantly reject the request and return the 500 status code you requested!

```json
{
  "error": "Internal Server Error",
  "message": "Chaos engineered error response",
  "status": 500
}
```

You can use this exact same endpoint to simulate `502 Bad Gateway` errors, `503 Service Unavailable` errors, or even `429 Too Many Requests` rate limiting errors!

### Wrapping Up

Robust test automation isn't just about proving that an application works; it is about proving that the application doesn't completely break when the infrastructure underneath it fails. 

By actively utilizing the `delay` and `status` query parameters, you can build highly resilient automation frameworks that are prepared for the unpredictable reality of the internet.

In the **next blog**, we will switch gears and explore the modern alternative to REST. We will learn how to write complex, nested queries using the mock server's built-in **GraphQL** endpoint!
