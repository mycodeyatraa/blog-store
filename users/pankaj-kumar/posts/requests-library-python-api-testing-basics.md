---
title: API Testing in Python: Introduction to the Requests Library
date: 01-Feb-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [python, api-testing, requests, json, backend, pytest]
category: API Testing
categories: [API Testing, Python]
excerpt: >-
  Step into the backend! Learn the fundamentals of API testing in Python using the legendary Requests library. Master GET requests, JSON parsing, and response validation.
readTime: 6 min read
---

# API Testing in Python: Introduction to the Requests Library

Welcome to **Phase 5: API Testing** in our Python Automation journey!

Up to this point, we have focused entirely on UI Automation using Selenium WebDriver. However, testing the UI is only half the battle. Modern web applications are powered by backend REST APIs. 

If you want to create user accounts, verify database states, or bypass tedious login screens during your UI tests, you absolutely must know how to automate APIs.

In the Python ecosystem, there is one library that rules them all: **`requests`**.

---

## 1. What is the Requests Library?

The `requests` library is the de facto standard for making HTTP requests in Python. It abstracts away the complexities of making network requests behind a beautiful, simple API.

To get started, install it via pip:

```bash
pip install requests
```

With just a few lines of code, you can trigger API calls and validate the backend without ever opening a browser!

---

## 2. Your First API Call (GET)

Let's make a simple HTTP `GET` request. A GET request is used to retrieve data from a server. We will use the free `reqres.in` API for our examples.

**tests/test_api_basics.py**

```python
import requests
def test_get_user_api():
    # 1. Define the Endpoint URL
    url = "https://reqres.in/api/users/2"
    # 2. Make the GET Request
    response = requests.get(url)
    # 3. Print out the raw data for debugging
    print(f"\nStatus Code: {response.status_code}")
    print(f"Response Body: {response.text}")
    # 4. Assert the Status Code is 200 (OK)
    assert response.status_code == 200
```

Notice how incredibly readable this is? There is no complex setup. You just call `requests.get()` and it returns a powerful `Response` object containing the status code, headers, and body!

---

## 3. Parsing JSON Responses

Most modern APIs return data in JSON format. The `requests` library has a built-in `.json()` method that automatically converts the JSON string payload into a native Python Dictionary!

Let's validate the data returned by the server:

```python
import requests
def test_validate_json_payload():
    url = "https://reqres.in/api/users/2"
    response = requests.get(url)
    # Convert response to a Python Dictionary
    json_data = response.json()
    print("\nParsed JSON:")
    print(json_data)
    # Validate specific fields using Python Dictionary syntax!
    assert json_data["data"]["id"] == 2
    assert json_data["data"]["email"] == "janet.weaver@reqres.in"
    assert json_data["data"]["first_name"] == "Janet"
```

Because Python dictionaries are so easy to navigate, validating complex, deeply nested JSON responses is a breeze.

---

## 4. Inspecting Headers and Timeouts

Professional API testing isn't just about validating the JSON body. You also need to validate HTTP Headers and ensure the API responds within an acceptable timeframe.

You can easily access Headers using the `response.headers` dictionary, and you can enforce timeouts directly in the request call!

```python
import requests
def test_api_headers_and_performance():
    url = "https://reqres.in/api/users/2"
    # Set a strict 2-second timeout. If the server is slow, the test will fail!
    response = requests.get(url, timeout=2.0)
    # Validate Headers
    content_type = response.headers.get("Content-Type")
    print(f"\nContent-Type is: {content_type}")
    assert "application/json" in content_type
    # Validate Performance (Response Time)
    elapsed_seconds = response.elapsed.total_seconds()
    print(f"API Responded in: {elapsed_seconds} seconds")
    assert elapsed_seconds < 1.0  # Fails if API takes more than 1 second
```

---

## 5. Execution Output

Let's run our new PyTest API suite. Because we are bypassing the UI, notice how unbelievably fast these tests execute!

```bash
pytest test_api_basics.py -s
```

```text
============================= test session starts ==============================
platform win32 -- Python 3.12.0, pytest-8.0.0
rootdir: C:\Automation\mycodeyatra-api
collected 3 items
test_api_basics.py 
Status Code: 200
Response Body: {"data":{"id":2,"email":"janet.weaver@reqres.in"...}}
.
Parsed JSON:
{'data': {'id': 2, 'email': 'janet.weaver@reqres.in', 'first_name': 'Janet', 'last_name': 'Weaver', 'avatar': 'https://reqres.in/img/faces/2-image.jpg'}, 'support': {'url': 'https://reqres.in/#support-heading', 'text': 'To keep ReqRes free, contributions towards server costs are appreciated!'}}
.
Content-Type is: application/json; charset=utf-8
API Responded in: 0.12564 seconds
.
============================== 3 passed in 0.52s ===============================
```

Three full end-to-end API tests executed in half a second! This is why shifting validation down to the API layer is critical for test scaling.

## Conclusion

The `requests` library is an absolute masterpiece of Python design. It makes interacting with HTTP APIs incredibly intuitive.
- Use `requests.get()` to retrieve data.
- Use `response.json()` to parse payloads into Python Dictionaries.
- Use `response.headers` and `response.elapsed` to validate server metadata and performance.

In our next article, we will go deeper into the `requests` library and explore how to validate complex arrays, handle query parameters, and test error codes using **Advanced GET APIs!**
