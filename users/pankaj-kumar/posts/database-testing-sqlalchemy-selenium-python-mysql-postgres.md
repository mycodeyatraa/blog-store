---
title: Enterprise Validation: UI + Database Testing with SQLAlchemy
date: 25-May-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [python, selenium, database, sqlalchemy, mysql, postgresql]
category: Enterprise Validation
categories: [Enterprise Validation, Python, Automation]
excerpt: >-
  Stop relying solely on UI assertions! Learn how to integrate SQLAlchemy with Selenium to seamlessly query, validate, and clean up MySQL and PostgreSQL databases during your automation suite execution.
readTime: 6 min read
---

# Enterprise Validation: UI + Database Testing with SQLAlchemy

A junior automation engineer only tests the UI. They write a Selenium script to fill out a registration form, click "Submit", and verify that a "Success" message appears on the screen.

But what if the UI says "Success", but the user was **never actually inserted into the database**? What if the UI says the purchase went through, but the payment record in the MySQL table shows a status of "Failed"?

In true Enterprise DevSecOps environments, UI validation is not enough. You must validate the underlying data layers. In this article, we will teach you how to integrate Python's powerful **SQLAlchemy** library alongside Selenium to seamlessly query and assert MySQL and PostgreSQL databases during your Pytest executions!

---

## 1. Why SQLAlchemy?

There are many Python libraries for interacting with databases (`psycopg2` for PostgreSQL, `mysql-connector` for MySQL). However, using individual drivers means you have to write raw, hardcoded SQL strings in your automation framework.

**SQLAlchemy** is an Object-Relational Mapper (ORM). It allows us to interact with *any* SQL database (MySQL, Postgres, SQLite) using unified Python objects. It abstracts away the complex SQL syntax, making our test code incredibly readable.

### Installation
You need SQLAlchemy and the specific driver for your database:

```bash
pip install sqlalchemy
pip install pymysql       # For MySQL
pip install psycopg2-binary # For PostgreSQL
```

---

## 2. Setting Up the Database Connection Fixture

We do not want to open and close a database connection for every single test. That would drastically slow down our suite. 

Instead, we will create a `db_session` Pytest fixture in our `conftest.py` file with `scope="session"`. This will open the database connection once when the Pytest suite begins, and share it across all tests!

**tests/conftest.py**

```python
import pytest
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker
@pytest.fixture(scope="session")
def db_session():
    # Connection String Format: dialect+driver://username:password@host:port/database
    # Example for MySQL:
    # db_url = "mysql+pymysql://admin:securepass@localhost:3306/mycodeyatra_db"
    # Example for PostgreSQL:
    db_url = "postgresql+psycopg2://admin:securepass@localhost:5432/mycodeyatra_db"
    print("\n[DB] Connecting to the database...")
    engine = create_engine(db_url)
    Session = sessionmaker(bind=engine)
    session = Session()
    yield session
    print("\n[DB] Closing database connection...")
    session.close()
    engine.dispose()
```

---

## 3. Creating the SQLAlchemy Model

To interact with a database table using SQLAlchemy, we define a standard Python class that maps to the table schema.

Let's assume we have a `users` table in our database.

**db_models/user.py**

```python
from sqlalchemy import Column, Integer, String
from sqlalchemy.ext.declarative import declarative_base
Base = declarative_base()
class User(Base):
    __tablename__ = 'users'
    id = Column(Integer, primary_key=True)
    username = Column(String)
    email = Column(String)
    status = Column(String)
```

---

## 4. The Hybrid Test: Selenium + Database Assertion

Now we write the ultimate Enterprise test case. We will use Selenium to interact with the frontend Registration page. Then, we will immediately use our `db_session` fixture to query the backend database and ensure the data was actually saved correctly!

**tests/test_registration.py**

```python
import time
from selenium.webdriver.common.by import By
from db_models.user import User
def test_user_registration_and_db_persistence(driver, db_session):
    # 1. UI ACTION: Register the user via the frontend
    driver.get("https://mycodeyatra.com/register")
    unique_username = f"testuser_{int(time.time())}"
    unique_email = f"{unique_username}@mycodeyatra.com"
    driver.find_element(By.ID, "username").send_keys(unique_username)
    driver.find_element(By.ID, "email").send_keys(unique_email)
    driver.find_element(By.ID, "password").send_keys("SecurePass123!")
    driver.find_element(By.ID, "submit-btn").click()
    # 2. UI ASSERTION: Verify the success message
    success_msg = driver.find_element(By.ID, "success-message").text
    assert "Registration successful" in success_msg
    # 3. DATABASE ASSERTION: Verify the user was created in the database!
    print(f"\n[DB] Querying database for username: {unique_username}")
    # Using SQLAlchemy to query the table without writing raw SQL!
    db_user = db_session.query(User).filter_by(username=unique_username).first()
    assert db_user is not None, f"User {unique_username} was NOT found in the database!"
    assert db_user.email == unique_email, "Stored email does not match UI input!"
    assert db_user.status == "ACTIVE", "User account was not set to ACTIVE!"
```

### Why is this powerful?
If a developer breaks the backend API but the frontend UI forgets to render the error, the UI will falsely report a "Success" message. A standard UI test would incorrectly pass. Our hybrid test will catch the bug because the `db_user` assertion will fail!

---

## 5. Teardown: Cleaning Up the Database

If your automation suite creates 500 fake users every time it runs, your database will eventually crash. You should always clean up the data you generate.

We can add a Pytest teardown block to our test to delete the user via SQLAlchemy:

```python
import pytest
from db_models.user import User
def test_user_registration_and_db_persistence(driver, db_session):
    # ... UI Actions and Assertions ...
    # Teardown logic
    yield 
    print(f"\n[DB] Cleaning up test data for {unique_username}...")
    db_user_to_delete = db_session.query(User).filter_by(username=unique_username).first()
    if db_user_to_delete:
        db_session.delete(db_user_to_delete)
        db_session.commit() # Don't forget to commit the deletion!
```

## Conclusion

Enterprise automation requires verifying the entire application stack.
- Use **SQLAlchemy** to connect your Pytest framework to MySQL, PostgreSQL, or SQLite.
- Create a reusable `db_session` fixture in `conftest.py`.
- Define Python models that map to your database tables.
- Combine **Selenium UI actions** with **SQLAlchemy database queries** in the exact same test script for ultimate validation!
- Always use teardown hooks to delete your test data and keep your environments clean.

In our next article, we will move beyond relational SQL databases and look at how to perform Data Validation on modern **NoSQL databases using MongoDB**!
