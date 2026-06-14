---
title: Automating POST APIs in Python: Creating New Resources
date: 05-Feb-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [python, api-testing, requests, post, json, faker]
category: API Testing
categories: [API Testing, Python]
excerpt: >-
  Stop testing with hardcoded data! Learn how to automate HTTP POST requests in Python, generate dynamic JSON payloads using the Faker library, and validate 201 Created server responses.
readTime: 6 min read
---

# Automating POST APIs in Python: Creating New Resources

Up to this point, we have been using `requests.get()` to retrieve existing data from the backend. 

But what if you are testing an e-commerce site and need to create a brand new Order? Or you are testing a CRM and need to create a brand new User? 

To create new data, we use the HTTP **POST** method. In this article, we will learn how to send JSON payloads to a server, generate dynamic test data using Python's `Faker` library, and validate that the resource was successfully created.

---

## 1. The Anatomy of a POST Request

Unlike a GET request, a POST request requires a **Payload** (or "Body"). This is the data you are sending to the server.

In Python, we simply define our JSON payload as a native Dictionary and pass it to the `json=` parameter of the `requests.post()` method!

**tests/test_post_api.py**

```python
import requests
def test_create_user_statically():
    url = "https://reqres.in/api/users"
    # 1. Define the Payload as a Python Dictionary
    payload = {
        "name": "Morpheus",
        "job": "Leader"
    }
    # 2. Make the POST Request using the `json` argument
    response = requests.post(url, json=payload)
    # 3. Print the response to see the newly created data
    print(f"\nStatus Code: {response.status_code}")
    print(f"Response JSON: {response.json()}")
    # 4. A successful POST request usually returns a 201 Created status!
    assert response.status_code == 201
```

If you execute this code, the API will respond with something like this:
`{'name': 'Morpheus', 'job': 'Leader', 'id': '451', 'createdAt': '2026-06-14T09:42:15.123Z'}`

Notice that the server automatically generated a unique `id` and a `createdAt` timestamp for us!

---

## 2. Dynamic Payloads with Faker

Hardcoding "Morpheus" is fine for a single test, but if you run this test 100 times, you will create 100 users named Morpheus. In enterprise automation, test data should always be dynamic.

We can use the `Faker` library to generate random names, emails, and job titles instantly!

First, install faker:

```bash
pip install Faker
```

Now, let's update our POST test to use dynamic data:

```python
import requests
from faker import Faker
def test_create_user_dynamically():
    url = "https://reqres.in/api/users"
    fake = Faker()
    # 1. Generate Fake Data
    random_name = fake.name()
    random_job = fake.job()
    print(f"\nAttempting to create User: {random_name} - {random_job}")
    # 2. Create the dynamic Payload
    payload = {
        "name": random_name,
        "job": random_job
    }
    # 3. Send the POST Request
    response = requests.post(url, json=payload)
    # 4. Assert the status is 201 Created
    assert response.status_code == 201
    # 5. Extract the auto-generated ID from the server response
    created_id = response.json()["id"]
    print(f"Success! Server generated User ID: {created_id}")
    # 6. Validate that the server correctly saved our random data
    assert response.json()["name"] == random_name
    assert response.json()["job"] == random_job
```

---

## 3. Form Data vs JSON Payloads

Sometimes, older APIs or File Upload endpoints do not accept JSON. They require `x-www-form-urlencoded` data (just like a standard HTML form submission).

If an API requires Form Data instead of JSON, you must use the `data=` argument instead of the `json=` argument!

```python
def test_post_form_data():
    url = "https://httpbin.org/post"
    form_payload = {
        "username": "admin",
        "password": "password123"
    }
    # Use data= for Form Submissions, NOT json=
    response = requests.post(url, data=form_payload)
    assert response.status_code == 200
```
*Note: Under the hood, using `data=` automatically sets the `Content-Type` header to `application/x-www-form-urlencoded`, while using `json=` automatically sets it to `application/json`.*

---

## 4. Execution Output

Let's run our dynamic POST test. You will see `Faker` generate a unique user, send it to the server, and the server respond with a brand new, auto-generated ID!

```bash
pytest test_post_api.py -s
```

```text
============================= test session starts ==============================
platform win32 -- Python 3.12.0, pytest-8.0.0
rootdir: C:\Automation\mycodeyatra-api
collected 2 items
test_post_api.py 
Status Code: 201
Response JSON: {'name': 'Morpheus', 'job': 'Leader', 'id': '912', 'createdAt': '2026-06-14T04:12:05.123Z'}
.
Attempting to create User: Jessica Davis - Data Scientist
Success! Server generated User ID: 845
.
============================== 2 passed in 0.85s ===============================
```

## Conclusion

Automating POST requests allows you to programmatically create data in your backend systems.
- Use the `json=payload` argument to send JSON data. The `requests` library will automatically serialize your Python dictionary into a JSON string and set the correct headers!
- Always validate that the response returns a `201 Created` status code.
- Use the `Faker` library to ensure you never hardcode test data. 

In our next article, we will learn how to modify and delete existing data using **PUT, PATCH, and DELETE APIs!**
