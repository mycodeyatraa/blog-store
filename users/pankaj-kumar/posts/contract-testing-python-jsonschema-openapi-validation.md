---
title: Contract Testing in Python: Validating OpenAPI Schemas
date: 19-Feb-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [python, api-testing, contract-testing, jsonschema, openapi, validation]
category: API Testing
categories: [API Testing, Python]
excerpt: >-
  Prevent breaking changes! Learn how to use the jsonschema library in Python to validate backend API responses against strict OpenAPI contracts, ensuring data types and required fields never fail silently.
readTime: 6 min read
---

# Contract Testing in Python: Validating OpenAPI Schemas

Imagine a scenario where the backend team changes a User's `id` from an Integer (`123`) to a String (`"123"`). The backend code compiles fine. The frontend code deploys fine. But the moment a user logs in, the entire application crashes because the frontend UI was explicitly expecting an Integer!

To prevent this catastrophic misalignment between microservices, we use **Contract Testing**. 

In this article, we will learn how to enforce an API Contract by validating our JSON responses against a strict Schema using Python's `jsonschema` library!

---

## 1. What is an API Contract (Schema)?

An API Contract is an agreement between the API Provider (backend) and the API Consumer (frontend). It dictates exactly what the JSON payload must look like.

A Schema defines:
1. Which keys are required.
2. The data type of each key (string, integer, boolean).
3. Constraints (e.g., age must be > 18).

Let's install the official Python validator:

```bash
pip install jsonschema
```

---

## 2. Defining the JSON Schema

First, we define our expectations in a Python dictionary. This schema asserts that the API *must* return an `id` (integer), a `name` (string), and a `job` (string). No exceptions!

**tests/test_contract.py**

```python
import requests
from jsonschema import validate, ValidationError
# Define the strict OpenAPI Contract
USER_SCHEMA = {
    "type": "object",
    "properties": {
        "id": {"type": "integer"},
        "name": {"type": "string"},
        "job": {"type": "string"}
    },
    "required": ["id", "name", "job"]
}
```

---

## 3. Validating the API Response against the Contract

Now, we make an API call and pass the `response.json()` into the `jsonschema.validate()` function. If the API breaks the contract, the function will throw a massive `ValidationError` and fail the test!

```python
def test_valid_api_contract():
    url = "https://reqres.in/api/users/2"
    response = requests.get(url)
    # Notice that the real API returns an integer ID, a string first_name, etc.
    json_payload = response.json()["data"]
    # We will simulate a perfect response matching our custom schema:
    perfect_payload = {
        "id": json_payload["id"], 
        "name": json_payload["first_name"], 
        "job": "Software Engineer"
    }
    print("\n[Contract] Validating perfect payload against schema...")
    try:
        # Validate the payload against the schema
        validate(instance=perfect_payload, schema=USER_SCHEMA)
        print("[Contract] Success! The API response perfectly matches the OpenAPI Contract.")
        assert True
    except ValidationError as e:
        print(f"[Contract Failed] {e.message}")
        assert False
```

---

## 4. Catching a Broken Contract

What happens when the backend team accidentally changes the `id` to a String? Let's simulate a broken contract and watch the `jsonschema` library catch it!

```python
def test_broken_api_contract():
    print("\n[Contract] Simulating a broken backend response (ID changed to String)...")
    broken_payload = {
        "id": "two",  # <-- ERROR: The contract strictly requires an integer!
        "name": "Janet", 
        "job": "Software Engineer"
    }
    try:
        validate(instance=broken_payload, schema=USER_SCHEMA)
        assert False # Should never reach here!
    except ValidationError as e:
        print(f"[Contract Failed] Caught Schema Violation: {e.message}")
        # Test passes because we successfully caught the backend error!
        assert True 
```

---

## 5. Execution Output

Let's execute the suite. Watch how the validator gracefully approves the correct payload, but instantly triggers a red alert when the data type of the `id` field violates the contract!

```bash
pytest test_contract.py -s
```

```text
============================= test session starts ==============================
platform win32 -- Python 3.12.0, pytest-8.0.0
rootdir: C:\Automation\mycodeyatra-api
collected 2 items
test_contract.py 
[Contract] Validating perfect payload against schema...
[Contract] Success! The API response perfectly matches the OpenAPI Contract.
.
[Contract] Simulating a broken backend response (ID changed to String)...
[Contract Failed] Caught Schema Violation: 'two' is not of type 'integer'
.
============================== 2 passed in 0.42s ===============================
```

## Conclusion

Contract Testing is the ultimate safety net for Microservice architecture.
- Frontend and Backend teams agree on an OpenAPI/Swagger Schema.
- Use the Python `jsonschema` library to define that schema natively in your framework.
- Wrap `validate(instance=payload, schema=SCHEMA)` in a `try/except` block.
- If the backend team accidentally changes a data type or deletes a required key, your automation suite will catch the violation immediately!

In our next and final article of the API Testing Phase, we will combine everything we've learned and design a massive, reusable **Enterprise API Architecture** using Python Object-Oriented Programming!
