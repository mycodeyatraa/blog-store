---
title: Token Validation: Decoding and Securing JWTs in Python Automation
date: 22-Mar-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [python, selenium, security, jwt, tokens, devsecops]
category: Security
categories: [Security, Python, DevSecOps]
excerpt: >-
  Are your JWTs leaking passwords? Learn how to decode Base64 JSON Web Tokens in Python without the secret key, and write automated Pytest security assertions to prevent sensitive PII leakage!
readTime: 6 min read
---

# Token Validation: Decoding and Securing JWTs in Python Automation

In Phase 6, we learned how to extract JSON Web Tokens (JWTs) from the browser's Local Storage to bypass login screens. 

But as a DevSecOps engineer, you must look closer. A **JWT is not encrypted**. It is only Base64 encoded! Anyone who possesses the token can easily decode the payload and read the data inside. 

If a developer accidentally embeds a user's plaintext password, Social Security Number, or sensitive Role IDs inside the JWT payload, it is a catastrophic PII (Personally Identifiable Information) breach. In this article, we will write automated security tests to decode JWTs and mathematically prove they do not leak sensitive data!

---

## 1. The Anatomy of a JWT

A JWT is a string consisting of three parts separated by periods (`.`):
`header.payload.signature`

1. **Header:** Contains metadata (like the signing algorithm, e.g., `HS256`).
2. **Payload:** Contains the actual claims (user ID, expiration time, email).
3. **Signature:** The cryptographic hash that prevents the payload from being tampered with.

Because the Payload is simply Base64 encoded JSON, we can decode it in Python without needing the server's private secret key!

---

## 2. Decoding a JWT in Python

To decode a JWT, we will install the official `PyJWT` library.

```bash
pip install PyJWT
```

Let's write a script that extracts a token and decodes its payload. Note that we will pass `options={"verify_signature": False}` because we only want to read the payload, not authenticate it!

**tests/test_jwt_security.py**

```python
import jwt
import json
def test_jwt_decoding():
    # 1. A simulated JWT token (Usually extracted from requests or Selenium)
    # This token's payload contains: {"user_id": 105, "role": "admin", "password": "mypassword123"}
    raw_token = (
        "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9."
        "eyJ1c2VyX2lkIjoxMDUsInJvbGUiOiJhZG1pbiIsInBhc3N3b3JkIjoibXlwYXNzd29yZDEyMyIsImV4cCI6MTUxNjIzOTAyMn0."
        "SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c"
    )
    print("\n[Security] Decoding JWT Payload...")
    # 2. Decode the payload without verifying the signature
    decoded_payload = jwt.decode(raw_token, options={"verify_signature": False})
    # Pretty print the decoded JSON
    print(json.dumps(decoded_payload, indent=2))
    assert "user_id" in decoded_payload
```

---

## 3. Writing JWT Security Assertions

Now that we have the payload as a Python Dictionary, we can write automated tests to enforce enterprise security rules.

1. **No Sensitive Data:** Ensure `password`, `ssn`, or `credit_card` keys do NOT exist.
2. **Expiration Enforcement:** Ensure the token has an `exp` (expiration) claim. A token without an expiration lives forever and is highly dangerous if stolen.
3. **Algorithm Check:** Ensure the header uses a secure algorithm like `HS256` or `RS256`, and not `none` (a famous JWT vulnerability).

Let's append these tests!

```python
import time
def test_jwt_no_sensitive_data_leakage():
    raw_token = get_live_token_from_api() # Imagine this fetches the live token
    decoded_payload = jwt.decode(raw_token, options={"verify_signature": False})
    # Security Rule 1: The payload must never contain a "password" key!
    assert "password" not in decoded_payload, "CRITICAL VULNERABILITY: JWT leaks plaintext password!"
    assert "ssn" not in decoded_payload, "CRITICAL VULNERABILITY: JWT leaks SSN!"
    print("\n✅ Passed: JWT does not leak sensitive PII.")
def test_jwt_expiration_exists_and_valid():
    raw_token = get_live_token_from_api()
    decoded_payload = jwt.decode(raw_token, options={"verify_signature": False})
    # Security Rule 2: Token must expire
    assert "exp" in decoded_payload, "CRITICAL VULNERABILITY: JWT has no expiration date!"
    # Ensure the expiration time is in the future
    current_time = int(time.time())
    assert decoded_payload["exp"] > current_time, "JWT is already expired!"
    print("✅ Passed: JWT has a valid expiration timestamp.")
def test_jwt_secure_algorithm():
    raw_token = get_live_token_from_api()
    # We can decode the unencrypted HEADER using jwt.get_unverified_header()
    header = jwt.get_unverified_header(raw_token)
    # Security Rule 3: The algorithm must never be "none"
    assert header["alg"] != "none", "CRITICAL VULNERABILITY: JWT accepts unverified 'none' algorithm!"
    assert header["alg"] in ["HS256", "RS256"], f"Insecure algorithm used: {header['alg']}"
    print("✅ Passed: JWT uses strong cryptographic algorithm.")
# Mock function for demonstration
def get_live_token_from_api():
    return (
        "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9."
        "eyJ1c2VyX2lkIjoxMDUsInJvbGUiOiJhZG1pbiIsImV4cCI6MjUxNjIzOTAyMn0." # Password removed! Future Exp!
        "signature"
    )
```

---

## 4. Execution Output

Let's execute our new DevSecOps suite!

```bash
pytest test_jwt_security.py -s
```

```text
============================= test session starts ==============================
platform win32 -- Python 3.12.0, pytest-8.0.0
rootdir: C:\Automation\mycodeyatra-security
collected 4 items
test_jwt_security.py 
[Security] Decoding JWT Payload...
{
  "user_id": 105,
  "role": "admin",
  "password": "mypassword123",
  "exp": 1516239022
}
.
✅ Passed: JWT does not leak sensitive PII.
.
✅ Passed: JWT has a valid expiration timestamp.
.
✅ Passed: JWT uses strong cryptographic algorithm.
.
============================== 4 passed in 0.85s ===============================
```

## Conclusion

Tokens are the keys to your application's kingdom.
- Because JWTs are Base64 encoded (not encrypted), you can easily extract them from your API tests and inspect the payload in Python.
- Use the `PyJWT` library to decode the token with `verify_signature: False`.
- Write automated tests to enforce that developers never embed passwords or PII in the payload, and always enforce strict expiration policies!

In our next article, we will move from static validation to active penetration testing! We will integrate **OWASP ZAP** with Python and Selenium to actively scan our application for vulnerabilities as our UI tests navigate the DOM!
