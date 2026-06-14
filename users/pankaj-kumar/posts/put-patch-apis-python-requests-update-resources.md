---
title: Automating PUT and PATCH APIs: Updating Resources in Python
date: 08-Feb-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [python, api-testing, requests, put, patch, update]
category: API Testing
categories: [API Testing, Python]
excerpt: >-
  Understand the critical difference between full updates (PUT) and partial updates (PATCH), and learn how to automate them seamlessly using Python's Requests library.
readTime: 6 min read
---

# Automating PUT and PATCH APIs: Updating Resources in Python

In previous articles, we used `GET` to read data and `POST` to create data. The next logical step in the API lifecycle is learning how to update existing records. 

In REST API architecture, there are two different HTTP methods used for updating data: **PUT** and **PATCH**. 

In this article, we will learn the exact difference between the two and how to automate them using Python's `requests` library.

---

## 1. PUT vs PATCH: What is the Difference?

Before writing code, you must understand the architectural difference:

- **PUT (Full Update):** You are completely replacing the existing resource. If a User has a `name`, `email`, and `job`, and you send a PUT request with only a `name`, the server will often delete the `email` and `job` fields because you didn't include them!
- **PATCH (Partial Update):** You are only updating specific fields. If you send a PATCH request with only a `name`, the server will update the name and leave the `email` and `job` untouched.

---

## 2. Automating a PUT Request

Let's assume we want to update User #2. We will completely overwrite their profile by sending a full JSON payload via `requests.put()`.

**tests/test_update_api.py**

```python
import requests
from faker import Faker
def test_put_full_update():
    # Target an existing user ID (User 2)
    url = "https://reqres.in/api/users/2"
    fake = Faker()
    new_name = fake.name()
    new_job = "Senior Engineer"
    # 1. Provide the FULL payload
    payload = {
        "name": new_name,
        "job": new_job
    }
    print(f"\n[PUT] Sending full update payload: {payload}")
    # 2. Make the PUT request
    response = requests.put(url, json=payload)
    # 3. Assert a successful 200 OK status
    assert response.status_code == 200
    # 4. Verify the server returned our updated data and a timestamp
    json_data = response.json()
    print(f"[PUT] Server Response: {json_data}")
    assert json_data["name"] == new_name
    assert json_data["job"] == new_job
    assert "updatedAt" in json_data
```

Notice that the server responds with a brand new `updatedAt` timestamp, proving our change was saved!

---

## 3. Automating a PATCH Request

Now, what if we only want to promote the user to "Manager", but we don't want to change their name or email? We should use a PATCH request.

To do this, we provide a partial JSON payload containing *only* the field we want to change, and use `requests.patch()`.

```python
import requests
def test_patch_partial_update():
    url = "https://reqres.in/api/users/2"
    # 1. Provide a PARTIAL payload
    payload = {
        "job": "Engineering Manager"
    }
    print(f"\n[PATCH] Sending partial update payload: {payload}")
    # 2. Make the PATCH request
    response = requests.patch(url, json=payload)
    assert response.status_code == 200
    json_data = response.json()
    print(f"[PATCH] Server Response: {json_data}")
    # 3. Validate the single field was updated
    assert json_data["job"] == "Engineering Manager"
    # Notice we didn't send a name, so the API handles it without overwriting it!
```

---

## 4. Execution Output

Let's execute both tests. Notice how the `PUT` request sends both fields, while the `PATCH` request seamlessly sends only the single modified field.

```bash
pytest test_update_api.py -s
```

```text
============================= test session starts ==============================
platform win32 -- Python 3.12.0, pytest-8.0.0
rootdir: C:\Automation\mycodeyatra-api
collected 2 items
test_update_api.py 
[PUT] Sending full update payload: {'name': 'David Miller', 'job': 'Senior Engineer'}
[PUT] Server Response: {'name': 'David Miller', 'job': 'Senior Engineer', 'updatedAt': '2026-06-14T04:14:22.451Z'}
.
[PATCH] Sending partial update payload: {'job': 'Engineering Manager'}
[PATCH] Server Response: {'job': 'Engineering Manager', 'updatedAt': '2026-06-14T04:14:23.112Z'}
.
============================== 2 passed in 0.82s ===============================
```

## Conclusion

Updating resources in Python is just as easy as creating them.
- Use `requests.put(url, json=payload)` when you intend to completely replace the existing database record. You must provide all mandatory fields.
- Use `requests.patch(url, json=payload)` when you want to execute a surgical strike on a single specific field (like changing an order status from "Pending" to "Shipped").
- Always validate that the response returns a `200 OK` and check the `updatedAt` timestamp to confirm the transaction went through!

In our next article, we will complete the CRUD lifecycle by learning how to permanently erase data from the database using **HTTP DELETE APIs!**
