---
title: Testing CRUD Operations
date: 21-Feb-2025
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [supertest, crud, post, get, put, delete, jest, typescript]
category: API Testing
categories: [API Testing, NodeJS, Automation, TypeScript]
excerpt: >-
  Master the core of API testing by writing an End-to-End Test Suite that sequentially validates the entire Create, Read, Update, and Delete lifecycle.
readTime: 6 min read
---

The true power of any API Testing framework lies in its ability to validate state changes across the entire lifecycle of a resource.

In API architecture, the fundamental operations for managing resources are **CREATE, READ, UPDATE, and DELETE** (commonly known as CRUD). These map directly to the HTTP methods `POST`, `GET`, `PUT/PATCH`, and `DELETE`.

In this tutorial, we will write a complete End-to-End (E2E) test suite using **SuperTest** and **Jest** to validate the full CRUD lifecycle of a User resource.

---

## 1. The Strategy: Chaining State

When writing an E2E CRUD test, the tests are strictly sequential.
1. We **POST** a user and capture the generated `userId` from the response.
2. We use that `userId` to **GET** the user and assert it exists.
3. We **PUT** an update to the user and verify the modification.
4. We **DELETE** the user.
5. We **GET** the user again and assert it returns a `404 Not Found`.

Let's implement this strategy in TypeScript!

## 2. Implementing the CRUD Suite

Create a file named `crud.test.ts`:

```typescript
import request from 'supertest';
describe('CRUD Operations with SuperTest', () => {
    const API_URL = 'http://localhost:8080';
    let userId: string; // Store state across tests
    it('1. CREATE (POST) a new user', async () => {
        const payload = {
            name: 'Pankaj Kumar',
            email: 'pankaj@mycodeyatra.com',
            role: 'Admin'
        };
        const response = await request(API_URL)
            .post('/api/users')
            .send(payload) // SuperTest automatically serializes to JSON!
            .set('Accept', 'application/json');
        // Assertions
        expect(response.status).toBe(201); // 201 Created
        expect(response.body).toHaveProperty('id');
        expect(response.body.name).toBe(payload.name);
        // Save the generated ID for the subsequent tests
        userId = response.body.id;
    });
    it('2. READ (GET) the created user', async () => {
        const response = await request(API_URL)
            .get(`/api/users/${userId}`);
        expect(response.status).toBe(200);
        expect(response.body.id).toBe(userId);
    });
    it('3. UPDATE (PUT) the user', async () => {
        const updatedPayload = {
            name: 'Pankaj (Updated)',
            email: 'pankaj@mycodeyatra.com',
            role: 'SuperAdmin'
        };
        const response = await request(API_URL)
            .put(`/api/users/${userId}`)
            .send(updatedPayload);
        expect(response.status).toBe(200);
        expect(response.body.role).toBe('SuperAdmin');
    });
    it('4. DELETE the user', async () => {
        const response = await request(API_URL)
            .delete(`/api/users/${userId}`);
        expect(response.status).toBe(204); // 204 No Content
    });
    it('5. Verify user is DELETED (GET)', async () => {
        const response = await request(API_URL)
            .get(`/api/users/${userId}`);
        // The user should no longer exist!
        expect(response.status).toBe(404);
    });
});
```

## 3. Key SuperTest Features Used

Notice how elegant the SuperTest API is:
* **`.send(payload)`**: SuperTest automatically serializes your JavaScript objects into a JSON string and sets the `Content-Type: application/json` header for you!
* **`.set('Header', 'Value')`**: The fluent interface allows you to easily inject custom HTTP Headers like Authorization tokens or Accept types.
* **`.status` and `.body`**: The response object comes perfectly parsed and ready for Jest's `expect()` assertions.

## 4. Conclusion

By organizing your tests logically, you have just validated the entire state machine of the backend API. 

However, passing raw JavaScript objects into `.send()` can become messy when testing APIs with massive, complex JSON payloads. In our next tutorial, we will explore how to supercharge our frameworks using **TypeScript Interfaces** to enforce strict typings on our payloads and responses!
