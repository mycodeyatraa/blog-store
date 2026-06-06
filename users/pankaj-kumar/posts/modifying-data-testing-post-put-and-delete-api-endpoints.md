---
title: Modifying Data: Testing POST, PUT, and DELETE API Endpoints
date: 2025-01-17
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [rest-api, api-testing, automation, crud, postman]
category: API Testing
categories: [API Testing, Automation, REST API]
excerpt: >-
  Real-world applications don't just read data; they constantly create, update, and delete records. In this post, we will explore the core methods used to modify data in REST architectures: POST, PUT, and DELETE.
readTime: 4 min read
---

In the previous tutorial, we learned how to read data from our server using the GET method. However, real-world applications don't just read data; they constantly create, update, and delete records.

In this post, we will explore the core methods used to modify data in REST architectures: POST, PUT, and DELETE. The MyCodeYatra Mock API Server fully supports these operations out-of-the-box!

> **Important Resource:** The entire codebase for this tutorial is hosted on our official GitHub Repository. You can find it here: [https://github.com/MYCodeYatra/myct-api-test-server](https://github.com/MYCodeYatra/myct-api-test-server)

---

## Creating Data (The POST Method)

When you submit a registration form on a website, the frontend typically sends a **POST** request to the server. This request contains a "body" or "payload" of JSON data representing the new record you want to create.

Let's simulate creating a brand new user in our mock database. To do this, we will send a POST request to the `/api/users` endpoint and attach a JSON body containing a name, email, and role.

Open your terminal and execute the following command:

```bash
curl -X POST http://localhost:8080/api/users \
-H "Content-Type: application/json" \
-d '{"name": "Test User", "email": "test@example.com", "role": "user"}'
```

If the request is successful, the server will respond with a `201 Created` status code. The response body will echo back the newly created user, and you will notice that the server automatically generated a unique ID and a timestamp for the new record!

```json
{
  "id": "a1b2c3d4-e5f6-7g8h-9i0j-k1l2m3n4o5p6",
  "name": "Test User",
  "email": "test@example.com",
  "role": "user",
  "createdAt": "2025-01-17T12:00:00.000Z"
}
```

---

## Updating Data (The PUT Method)

Now that we have created a user, what happens if they want to change their email address? In REST, we update existing records using the **PUT** (or PATCH) method.

To update a record, you must specify the exact ID of the user you want to modify in the URL path, just like we learned in the previous blog. Then, you pass the updated data in the JSON body.

Using the ID that the server just generated for your test user, run this update command in your terminal:

```bash
curl -X PUT http://localhost:8080/api/users/YOUR-GENERATED-ID \
-H "Content-Type: application/json" \
-d '{"name": "Test User Updated", "email": "new.email@example.com", "role": "admin"}'
```

The server will process the update and return a `200 OK` status code. The response will show your modified data, confirming that the user has been successfully promoted to an admin role!

---

## Deleting Data (The DELETE Method)

Finally, we need a way to completely remove records from the database. This is handled by the **DELETE** method. 

Unlike POST and PUT, the DELETE method rarely requires a JSON body. The server only needs to know the ID of the record it should destroy, which we provide directly in the URL path.

To permanently delete your test user from the mock database, run this final command:

```bash
curl -X DELETE http://localhost:8080/api/users/YOUR-GENERATED-ID
```

The server will return a `200 OK` (or sometimes a `204 No Content` in strict REST environments) indicating that the deletion was successful. 

If you try to make a GET request to fetch that exact same user ID again, you will correctly receive a `404 Not Found` error. The data is gone forever!

### Wrapping Up

You now have complete mastery over the core CRUD (Create, Read, Update, Delete) operations! 

By sending JSON payloads via POST and PUT, and managing records via DELETE, you can write powerful, end-to-end automation scripts that simulate real user workflows.

In the **next blog**, we will tackle one of the most notoriously tricky aspects of API testing: Pagination and Sorting!
