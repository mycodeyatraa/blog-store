---
title: Advanced Validation for GET APIs with Python Requests
date: 03-Feb-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [python, api-testing, requests, json, validation, pytest]
category: API Testing
categories: [API Testing, Python]
excerpt: >-
  Go beyond the basics! Master Python list comprehensions to validate JSON arrays, learn how to handle URL query parameters dynamically, and write rock-solid negative tests for 404 error codes.
readTime: 6 min read
---

# Advanced Validation for GET APIs with Python Requests

In the previous article, we learned how to make a basic GET request and validate a single JSON object. But in the real world, APIs are far more complex. 

APIs often return massive Lists (Arrays) of data. They require Query Parameters to filter results. They also purposefully return Error Codes (like 404 or 500) when things go wrong, which we must validate.

In this article, we will master advanced validation techniques for HTTP GET requests!

---

## 1. Handling Query Parameters

If you search for shoes on Amazon, the URL might look like this: `https://amazon.com/api/products?category=shoes&sort=price_desc`. 

Those values after the `?` are called **Query Parameters**. 

Instead of hardcoding them into the URL string, the `requests` library allows us to pass them securely using a Python dictionary!

**tests/test_get_queries.py**

```python
import requests
def test_get_users_with_query_params():
    url = "https://reqres.in/api/users"
    # 1. Define your Query Parameters as a Dictionary
    payload = {
        "page": 2,
        "per_page": 3
    }
    # 2. Pass them to the `params` argument
    response = requests.get(url, params=payload)
    # Notice that requests automatically formats the URL for us!
    print(f"\nFinal URL executed: {response.url}")
    json_data = response.json()
    assert response.status_code == 200
    # 3. Validate the API correctly applied the pagination filters
    assert json_data["page"] == 2
    assert json_data["per_page"] == 3
```

---

## 2. Validating JSON Arrays (Lists)

When an API returns a list of items (like a list of users), the JSON response contains an array. In Python, a JSON array becomes a native `list`.

Let's validate that our API returned exactly 3 users, and check that *at least one* of them has the email "george.edwards@reqres.in".

```python
import requests
def test_validate_json_array():
    url = "https://reqres.in/api/users"
    response = requests.get(url, params={"page": 2})
    json_data = response.json()
    # 1. Extract the List of users (The "data" key holds the array)
    users_list = json_data["data"]
    # 2. Assert the length of the list
    print(f"\nTotal users returned: {len(users_list)}")
    assert len(users_list) == 6
    # 3. Search through the list using Python list comprehensions!
    emails = [user["email"] for user in users_list]
    print(f"Emails found: {emails}")
    # 4. Assert specific data exists inside the array
    assert "george.edwards@reqres.in" in emails
    assert "rachel.howell@reqres.in" in emails
```

Python list comprehensions `[user["email"] for user in users_list]` make extracting data from large JSON arrays incredibly fast and readable!

---

## 3. Validating Negative Scenarios (404 Not Found)

Testing the "Happy Path" (200 OK) is easy. But a good QA Engineer must also test the "Negative Paths". 

What happens if we request a User ID that does not exist in the database? The API should return a `404 Not Found` error. If it returns a 500 Server Error or a 200 OK, the API is broken!

```python
import requests
def test_get_user_not_found():
    # User ID 999999 does not exist
    url = "https://reqres.in/api/users/999999"
    response = requests.get(url)
    print(f"\nExpected 404, Actual: {response.status_code}")
    print(f"Response Body: {response.text}")
    # Assert that the API gracefully handled the missing resource
    assert response.status_code == 404
    # Often, APIs return an empty body `{}` on 404
    assert response.json() == {}
```

---

## 4. Execution Output

Let's run our test file. Notice how we seamlessly validated pagination, parsed an array of emails using list comprehensions, and successfully verified a negative 404 scenario!

```bash
pytest test_get_queries.py -s
```

```text
============================= test session starts ==============================
platform win32 -- Python 3.12.0, pytest-8.0.0
rootdir: C:\Automation\mycodeyatra-api
collected 3 items
test_get_queries.py 
Final URL executed: https://reqres.in/api/users?page=2&per_page=3
.
Total users returned: 6
Emails found: ['michael.lawson@reqres.in', 'lindsay.ferguson@reqres.in', 'tobias.funke@reqres.in', 'byron.fields@reqres.in', 'george.edwards@reqres.in', 'rachel.howell@reqres.in']
.
Expected 404, Actual: 404
Response Body: {}
.
============================== 3 passed in 0.77s ===============================
```

## Conclusion

Mastering HTTP GET requests requires you to handle data dynamically. 
- Always pass `params={}` as a dictionary rather than hardcoding long URLs.
- Use Python's powerful **List Comprehensions** to instantly extract specific fields from massive JSON arrays.
- Always write Negative Tests to ensure your API returns the correct `400` or `404` status codes when invalid data is provided.

In our next article, we will move beyond retrieving data and learn how to create brand new data in the database using **HTTP POST APIs!**
