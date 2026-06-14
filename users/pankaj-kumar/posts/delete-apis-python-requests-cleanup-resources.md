---
title: Automating DELETE APIs: Cleaning Up Resources in Python
date: 10-Feb-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [python, api-testing, requests, delete, crud, pytest]
category: API Testing
categories: [API Testing, Python]
excerpt: >-
  Complete the CRUD lifecycle! Learn how to safely execute HTTP DELETE requests, validate 204 No Content status codes, and write a full end-to-end API test in Python.
readTime: 6 min read
---

# Automating DELETE APIs: Cleaning Up Resources in Python

Welcome to the final stage of the CRUD lifecycle!

We have successfully automated the creation (POST), reading (GET), and updating (PUT/PATCH) of data via REST APIs using Python. However, if your automation suite creates 1,000 users every night and never deletes them, your database will eventually crash.

A professional automation framework must always clean up after itself. In this article, we will learn how to permanently erase data using the HTTP **DELETE** method.

---

## 1. The Anatomy of a DELETE Request

Unlike `POST` or `PUT`, a `DELETE` request rarely requires a JSON payload. Instead, you simply specify the unique ID of the resource you wish to delete directly in the URL endpoint.

**tests/test_delete_api.py**

```python
import requests
def test_delete_user():
    # Target an existing user ID (User 2)
    url = "https://reqres.in/api/users/2"
    print(f"\n[DELETE] Attempting to erase resource at {url}")
    # 1. Make the DELETE request
    response = requests.delete(url)
    # 2. Validate the Status Code
    print(f"[DELETE] Server responded with Status Code: {response.status_code}")
    # 204 No Content is the standard success code for a DELETE operation!
    assert response.status_code == 204
    # 3. Assert the response body is empty (No Content!)
    assert response.text == ""
```

Notice the status code! When you create data, the server returns `201 Created`. When you read data, it returns `200 OK`. But when you successfully delete data, a properly designed API will return **`204 No Content`**. This simply means "The operation succeeded, and I have nothing left to send back to you."

---

## 2. End-to-End CRUD Validation

Now that we know all four HTTP verbs, let's write a complete End-to-End API test. 

We will:
1. Create a User (POST)
2. Read the User (GET)
3. Update the User (PUT)
4. Delete the User (DELETE)
5. Verify the User is gone (GET -> 404)

```python
import requests
from faker import Faker
def test_end_to_end_crud_lifecycle():
    base_url = "https://reqres.in/api/users"
    fake = Faker()
    # --- 1. CREATE (POST) ---
    post_payload = {"name": fake.name(), "job": "QA Tester"}
    post_response = requests.post(base_url, json=post_payload)
    assert post_response.status_code == 201
    # Extract the new User ID
    user_id = post_response.json()["id"]
    user_url = f"{base_url}/{user_id}"
    print(f"\n[POST] Created User ID: {user_id}")
    # --- 2. UPDATE (PUT) ---
    put_payload = {"name": fake.name(), "job": "QA Lead"}
    put_response = requests.put(user_url, json=put_payload)
    assert put_response.status_code == 200
    print(f"[PUT] Updated Job to: {put_response.json()['job']}")
    # --- 3. DELETE (DELETE) ---
    delete_response = requests.delete(user_url)
    assert delete_response.status_code == 204
    print("[DELETE] Successfully erased the user!")
```

*Note: Since `reqres.in` is a mocked API, a subsequent GET request to the newly created ID will not actually return a 404, as the data isn't permanently persisted on their public server. In a real database environment, you would add a final `GET` assertion to verify the 404.*

---

## 3. Execution Output

Let's run our test. Watch how fast Python executes a complete CRUD lifecycle spanning four different HTTP verbs!

```bash
pytest test_delete_api.py -s
```

```text
============================= test session starts ==============================
platform win32 -- Python 3.12.0, pytest-8.0.0
rootdir: C:\Automation\mycodeyatra-api
collected 2 items
test_delete_api.py 
[DELETE] Attempting to erase resource at https://reqres.in/api/users/2
[DELETE] Server responded with Status Code: 204
.
[POST] Created User ID: 312
[PUT] Updated Job to: QA Lead
[DELETE] Successfully erased the user!
.
============================== 2 passed in 1.45s ===============================
```

## Conclusion

The ability to clean up after your automation suite is paramount.
- Use `requests.delete(url)` to permanently erase a resource.
- Always target the specific resource ID in the URL path (e.g., `/users/5`).
- Ensure you validate the `204 No Content` status code.
- If possible, execute a final `GET` request to verify the server correctly returns a `404 Not Found`.

In our next article, we will move into Advanced API concepts. We will explore how to automate secure APIs that require JWT tokens and OAuth 2.0 using the Requests library!
