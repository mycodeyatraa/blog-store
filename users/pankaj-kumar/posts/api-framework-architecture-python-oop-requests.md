---
title: Architecting an Enterprise API Framework in Python
date: 21-Feb-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [python, api-testing, framework, architecture, oop, requests]
category: API Testing
categories: [API Testing, Python]
excerpt: >-
  Design a scalable API testing framework! Learn how to use Object-Oriented Programming (OOP) in Python to build a centralized API Client, domain-specific services, and automated logging for your Requests suite.
readTime: 6 min read
---

# Architecting an Enterprise API Framework in Python

Congratulations! You have mastered the Python `requests` library. You know how to execute GET, POST, PUT, and DELETE methods. You know how to extract JWTs, validate JSON Schemas, and bypass UI logins using Hybrid Testing.

However, writing `requests.get()` inside every single test file is a terrible practice. If the base URL changes, or the authentication mechanism changes, you will have to update 500 different test files.

In this final article of our API Testing phase, we will encapsulate all of our API logic into an Object-Oriented **API Client Architecture**.

---

## 1. The Core API Client

The heart of an API Framework is the `APIClient` class. This class uses a single `requests.Session()` object, manages the Base URL, automatically injects Authentication tokens, and wraps all CRUD methods to automatically log execution data!

**utils/api_client.py**

```python
import requests
import logging
class APIClient:
    def __init__(self, base_url):
        self.base_url = base_url
        self.session = requests.Session()
        # Setup basic logging for the API Client
        logging.basicConfig(level=logging.INFO)
        self.logger = logging.getLogger("APIClient")
    def authenticate(self, username, password):
        """Authenticates and attaches the token to all future requests."""
        self.logger.info(f"Authenticating as {username}...")
        # Pretend we hit a login endpoint that returns a token
        payload = {"email": username, "password": password}
        response = self.session.post(f"{self.base_url}/login", json=payload)
        if response.status_code == 200:
            token = response.json().get("token")
            self.session.headers.update({"Authorization": f"Bearer {token}"})
            self.logger.info("Authentication Successful! Token attached to Session.")
        else:
            self.logger.error("Authentication Failed!")
            raise Exception("Login Failed")
    def get(self, endpoint, params=None):
        self.logger.info(f"GET Request to {endpoint}")
        return self.session.get(f"{self.base_url}{endpoint}", params=params)
    def post(self, endpoint, payload):
        self.logger.info(f"POST Request to {endpoint} with data: {payload}")
        return self.session.post(f"{self.base_url}{endpoint}", json=payload)
```

By wrapping `self.session.get()` inside a custom `get()` method, we instantly added centralized logging to our entire framework! 

---

## 2. Creating Domain-Specific Services

Now that we have a core `APIClient`, we shouldn't use it directly in our tests. Instead, we create **Service Classes** (just like Page Objects in UI testing!).

**services/user_service.py**

```python
class UserService:
    def __init__(self, api_client):
        self.api = api_client
        self.endpoint = "/users"
    def get_all_users(self, page=1):
        return self.api.get(self.endpoint, params={"page": page})
    def create_user(self, name, job):
        payload = {"name": name, "job": job}
        return self.api.post(self.endpoint, payload)
```

Notice how clean this is? The `UserService` knows *what* the endpoints and payloads are, but it delegates the actual HTTP networking to the `APIClient`. 

---

## 3. The Final Test Implementation

Let's write an End-to-End test using our new Architecture. We will initialize the Client, pass it to the Service, and execute our tests.

**tests/test_enterprise_api.py**

```python
from utils.api_client import APIClient
from services.user_service import UserService
def test_enterprise_framework_architecture():
    # 1. Initialize the Core Client
    client = APIClient(base_url="https://reqres.in/api")
    # 2. Authenticate the Session globally
    client.authenticate("eve.holt@reqres.in", "cityslicka")
    # 3. Initialize Domain Services
    user_service = UserService(client)
    # 4. Execute Tests cleanly!
    print("\n--- Executing Tests ---")
    # Test GET
    get_response = user_service.get_all_users(page=2)
    assert get_response.status_code == 200
    print(f"Retrieved {len(get_response.json()['data'])} users from page 2.")
    # Test POST
    post_response = user_service.create_user("Neo", "The One")
    assert post_response.status_code == 201
    print(f"Created User ID: {post_response.json()['id']}")
```

---

## 4. Execution Output

Let's execute the suite. Pay close attention to the console output: our `APIClient` automatically logs every single HTTP transaction, while our `UserService` executes the business logic flawlessly!

```bash
pytest test_enterprise_api.py -s
```

```text
============================= test session starts ==============================
platform win32 -- Python 3.12.0, pytest-8.0.0
rootdir: C:\Automation\mycodeyatra-api
collected 1 item
test_enterprise_api.py 
INFO:APIClient:Authenticating as eve.holt@reqres.in...
INFO:APIClient:Authentication Successful! Token attached to Session.
--- Executing Tests ---
INFO:APIClient:GET Request to /users
Retrieved 6 users from page 2.
INFO:APIClient:POST Request to /users with data: {'name': 'Neo', 'job': 'The One'}
Created User ID: 412
.
============================== 1 passed in 1.25s ===============================
```

## Phase 5 Complete!

By utilizing Object-Oriented Programming, we have decoupled our HTTP Networking (`APIClient`), our Business Endpoints (`UserService`), and our Assertions (`test_enterprise_api.py`). This is the exact architecture used by FAANG companies to scale API suites to millions of daily executions.

You have officially conquered **Phase 5: API Testing**.

In the next phase of our Python Curriculum, we will dive into one of the most critical topics in modern automation: **Phase 6: Authentication & Security!** We will learn how to bypass Single Sign-On (SSO), automate Multi-Factor Authentication (MFA), and execute deep cookie manipulation!
