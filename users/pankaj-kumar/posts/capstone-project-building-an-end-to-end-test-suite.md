---
title: Capstone Project: Building an End-to-End Test Suite
date: 27-Jan-2025
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [capstone, api-testing, automation, end-to-end, project]
category: API Testing
categories: [API Testing, Automation, Project]
excerpt: >-
  It is now time to prove everything you have learned. In this final Capstone Project, you will build a comprehensive test suite that touches every feature of the mock server in a single execution flow.
readTime: 6 min read
---

Welcome to the final blog in the MyCodeYatra Mock API Server Mastery series! 

Over the past thirteen tutorials, we have deconstructed every single component of modern API testing. We have mastered REST methods, pagination, Docker containers, Chaos Engineering, GraphQL queries, WebSocket streams, asynchronous Webhooks, and GitHub Actions.

It is now time to prove everything you have learned. In this final Capstone Project, you will build a comprehensive, automated test suite that touches every single feature of the mock server in a single, unified execution flow.

> **Important Resource:** The entire codebase for this tutorial is hosted on our official GitHub Repository. You can find it here: [https://github.com/MYCodeYatra/myct-api-test-server](https://github.com/MYCodeYatra/myct-api-test-server)

---

## The Capstone Scenario

You have been hired as the Lead QA Automation Engineer for a new e-commerce startup. The backend team has just handed you the MyCodeYatra Mock API Server, which they claim is "production-ready". 

Your job is to write a single automation script (using Postman, RestAssured, Cypress, Playwright, or your tool of choice) that executes the following end-to-end workflow:

**Phase 1: Setup & Security**
The script must ping the `/api/health` endpoint to ensure the server is alive. It must then send a POST request to `/api/auth/login` with valid credentials, capture the resulting JWT token, and store it in a variable for all future secure requests.

**Phase 2: REST Data Manipulation**
Using the captured JWT token, the script must create a new product using a POST request to `/api/products`. It must then use a PUT request to update the product's price, and finally use a GET request with pagination query parameters (`?page=1&limit=5`) to verify the product appears in the paginated catalog.

**Phase 3: Chaos Validation**
The script must send a GET request to `/api/users?delay=3000` to simulate latency. Your testing framework must successfully wait for the 3-second delay and assert that the correct data eventually arrives without timing out.

**Phase 4: GraphQL Integration**
The script must execute a GraphQL Mutation using variables to create a new User. Immediately afterward, it must execute a GraphQL Query to fetch that exact user, ensuring that no excess data (like the createdAt timestamp) is over-fetched.

**Phase 5: Cleanup**
Finally, the script must send a DELETE request to permanently remove the product and the user created during the test, ensuring the database is perfectly clean for the next run.

---

## Your Homework

This is your final challenge! We have provided the exact requirements, but we are not going to provide the code. It is entirely up to you to pick your favorite testing framework and build this script from scratch.

When you finish, you will possess a portfolio-grade automation script that proves you understand how to navigate modern, complex backend architectures.

### Thank You!

From the entire team at MyCodeYatra, thank you for joining us on this incredible 14-part journey. You are no longer just a tester; you are an API Automation Architect. 

Keep exploring, keep breaking things, and most importantly, keep coding!
