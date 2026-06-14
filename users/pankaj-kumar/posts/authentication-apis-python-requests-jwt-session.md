---
title: Automating Authenticated APIs: JWTs and Requests Sessions in Python
date: 12-Feb-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [python, api-testing, requests, authentication, jwt, session]
category: API Testing
categories: [API Testing, Python]
excerpt: >-
  Unlock secure APIs! Learn how to extract JSON Web Tokens (JWT) from login endpoints, manage HTTP Basic Auth, and elegantly persist security headers using Python's requests.Session().
readTime: 6 min read
---

# Automating Authenticated APIs: JWTs and Requests Sessions in Python

So far, we have been testing public APIs (`reqres.in`) that do not require authentication. In a corporate environment, 99% of the APIs you test will be locked behind stringent security measures like JWT (JSON Web Tokens) or OAuth 2.0.

If you attempt to send a `GET` request to a secure endpoint without a token, the server will immediately reject you with a `401 Unauthorized` or `403 Forbidden` error.

In this article, we will learn how to log in via an API, extract the JWT security token, and dynamically inject it into all subsequent requests using Python's powerful `requests.Session()` object!

---

## 1. The Manual Way: Injecting Headers

When an API uses Bearer Token authentication (JWT), you must pass the token inside the HTTP `Authorization` header.

Let's assume we have an API endpoint (`/login`) that returns a token when we provide valid credentials. We can extract that token and manually pass it to the next request via the `headers` argument.

**tests/test_auth_api.py**

```python
import requests
def test_manual_jwt_authentication():
    # 1. Authenticate to get the token
    login_url = "https://reqres.in/api/login"
    credentials = {
        "email": "eve.holt@reqres.in",
        "password": "cityslicka"
    }
    login_response = requests.post(login_url, json=credentials)
    assert login_response.status_code == 200
    # Extract the token from the JSON payload
    token = login_response.json()["token"]
    print(f"\n[Login] Extracted Token: {token}")
    # 2. Access a Secure Endpoint
    secure_url = "https://reqres.in/api/users/2" # Pretend this is secure
    # We must manually construct the Authorization header
    auth_headers = {
        "Authorization": f"Bearer {token}",
        "Content-Type": "application/json"
    }
    # Pass the headers dictionary into the GET request
    secure_response = requests.get(secure_url, headers=auth_headers)
    assert secure_response.status_code == 200
    print("[Secure API] Successfully accessed locked data!")
```

This works, but manually passing `headers=auth_headers` to every single request in your framework violates the DRY principle. There is a much better way.

---

## 2. The Professional Way: Requests Sessions

The `requests` library provides a brilliant feature called a **Session Object**. 

A Session persists parameters (like cookies and headers) across multiple requests. If we attach our JWT token to the Session once, every single `GET`, `POST`, or `PUT` request made using that session will automatically carry the token!

```python
import requests
def test_session_authentication():
    # 1. Create a global Session object
    api_session = requests.Session()
    # 2. Login to get the token
    login_response = api_session.post(
        "https://reqres.in/api/login", 
        json={"email": "eve.holt@reqres.in", "password": "cityslicka"}
    )
    token = login_response.json()["token"]
    # 3. Attach the token to the Session globally!
    api_session.headers.update({"Authorization": f"Bearer {token}"})
    print(f"\n[Session] Headers globally updated with token: Bearer {token}")
    # 4. Now, make requests WITHOUT explicitly passing the headers!
    response_1 = api_session.get("https://reqres.in/api/users/2")
    response_2 = api_session.get("https://reqres.in/api/users/3")
    assert response_1.status_code == 200
    assert response_2.status_code == 200
    print("[Session] Automatically authenticated multiple requests!")
```

By using `api_session = requests.Session()`, you eliminate the need to constantly pass headers back and forth. This is the industry standard for Python API frameworks!

---

## 3. Basic Authentication

Sometimes, older legacy APIs do not use JWT tokens. Instead, they use HTTP Basic Authentication (a username and password encoded in Base64). 

The `requests` library makes this incredibly simple. You don't need to manually encode anything. You just pass an `auth=(user, pass)` tuple directly into the request!

```python
import requests
def test_http_basic_auth():
    url = "https://httpbin.org/basic-auth/admin/password123"
    # The 'auth' tuple automatically handles the Base64 encoding!
    response = requests.get(url, auth=("admin", "password123"))
    print(f"\n[Basic Auth] Server Response: {response.json()}")
    assert response.status_code == 200
    assert response.json()["authenticated"] == True
```

---

## 4. Execution Output

Let's execute all three authentication strategies! Notice how Python seamlessly handles JWT extraction, Session-level header injection, and HTTP Basic Base64 encoding completely under the hood.

```bash
pytest test_auth_api.py -s
```

```text
============================= test session starts ==============================
platform win32 -- Python 3.12.0, pytest-8.0.0
rootdir: C:\Automation\mycodeyatra-api
collected 3 items
test_auth_api.py 
[Login] Extracted Token: QpwL5tke4Pnpja7X4
[Secure API] Successfully accessed locked data!
.
[Session] Headers globally updated with token: Bearer QpwL5tke4Pnpja7X4
[Session] Automatically authenticated multiple requests!
.
[Basic Auth] Server Response: {'authenticated': True, 'user': 'admin'}
.
============================== 3 passed in 1.10s ===============================
```

## Conclusion

Automating API security is a critical skill for any SDET.
- To handle modern **JWT / OAuth 2.0**, extract the token from the login response and attach it to an `Authorization: Bearer <token>` header.
- Always use `requests.Session()` to persist headers and cookies across multiple requests, keeping your code incredibly clean.
- To handle legacy **Basic Authentication**, simply use the `auth=("user", "pass")` tuple provided natively by Python!

In our next article, we will blow your mind by combining everything we've learned in the last 40 articles. We will create a **Hybrid Testing Framework** that uses API calls to bypass UI login screens in Selenium!
