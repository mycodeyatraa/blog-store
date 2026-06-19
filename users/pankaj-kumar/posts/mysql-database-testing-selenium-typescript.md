---
title: Validating SQL: MySQL Testing with TypeScript
date: 18-Jan-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [typescript, selenium, mysql, database-testing, enterprise-validation, sql, backend]
category: Selenium TypeScript
categories: [Selenium TypeScript, Enterprise Validation]
excerpt: >-
  Connect your automation framework directly to a relational database. Learn how to securely query MySQL databases, build Data Access Objects (DAOs), and verify backend data insertions after UI interactions.
readTime: 4 min read
---

# Validating SQL: MySQL Testing with TypeScript

In our previous tutorial, we established the core principle of Enterprise Validation: you cannot trust the UI. You must verify the underlying data layer.

Today, we are going to implement this strategy by connecting our Selenium TypeScript framework directly to a **MySQL** database.

MySQL is one of the most widely used relational databases in the world. When a user submits a registration form on your UI, that data almost certainly ends up in a MySQL (or PostgreSQL) table. 

Let's write a test that clicks "Submit" in the browser and then verifies the SQL insertion!

---

## 1. Installing the MySQL Driver

We will not use a heavy ORM like TypeORM for this simple demonstration. Instead, we will use the native `mysql2` driver, which provides a fast, Promise-based API for Node.js.

Install the driver via npm:

```bash
npm install mysql2
npm install --save-dev @types/node
```

---

## 2. Setting Up the Database Configuration

Remember the Golden Rule of DevOps: **Never hardcode credentials.**

Create a `.env` file at the root of your project:

```env
DB_HOST=localhost
DB_USER=qa_automation
DB_PASSWORD=secret_qa_pass
DB_NAME=enterprise_app_db
```

Now, create a dedicated `DatabaseManager.ts` file in your `src/utils` folder. This class will handle the connection logic securely:

```typescript
import mysql from 'mysql2/promise';
import * as dotenv from 'dotenv';
dotenv.config();
export class DatabaseManager {
  private static connection: mysql.Connection | null = null;
  static async getConnection(): Promise<mysql.Connection> {
    if (!this.connection) {
      this.connection = await mysql.createConnection({
        host: process.env.DB_HOST,
        user: process.env.DB_USER,
        password: process.env.DB_PASSWORD,
        database: process.env.DB_NAME,
      });
      console.log('[DB] Connected to MySQL successfully.');
    }
    return this.connection;
  }
  static async closeConnection() {
    if (this.connection) {
      await this.connection.end();
      this.connection = null;
      console.log('[DB] MySQL connection closed.');
    }
  }
}
```

---

## 3. Creating the Data Access Object (DAO)

We never write SQL queries directly inside our test files. We abstract them into a Data Access Object (DAO).

Create `UserDAO.ts` in your `src/db` folder:

```typescript
import { DatabaseManager } from '../utils/DatabaseManager';
import { RowDataPacket } from 'mysql2';
export class UserDAO {
  static async getUserByEmail(email: string) {
    const db = await DatabaseManager.getConnection();
    // Using parameterized queries (?) prevents SQL Injection!
    const query = 'SELECT * FROM users WHERE email = ? LIMIT 1';
    // Execute the query
    const [rows] = await db.execute<RowDataPacket[]>(query, [email]);
    if (rows.length === 0) return null;
    return rows[0]; // Return the raw database object
  }
  static async deleteUserByEmail(email: string) {
    const db = await DatabaseManager.getConnection();
    const query = 'DELETE FROM users WHERE email = ?';
    await db.execute(query, [email]);
  }
}
```

---

## 4. The Enterprise Test

Now, we combine Selenium and MySQL in our test execution layer. 

Notice how we use the database in the `afterAll` hook to clean up our test data. This ensures our tests are completely stateless and repeatable!

```typescript
import { Builder, WebDriver } from 'selenium-webdriver';
import { expect } from 'chai';
import { RegistrationPage } from '../pages/RegistrationPage';
import { UserDAO } from '../db/UserDAO';
import { DatabaseManager } from '../utils/DatabaseManager';
describe('User Registration E2E Flow', function () {
  let driver: WebDriver;
  const testEmail = 'auto_test_123@mycodeyatra.com';
  before(async function () {
    driver = await new Builder().forBrowser('chrome').build();
  });
  after(async function () {
    await driver.quit();
    // Teardown: Delete the user we just created so the test can run again!
    await UserDAO.deleteUserByEmail(testEmail);
    await DatabaseManager.closeConnection();
  });
  it('should register a new user and verify DB insertion', async function () {
    // 1. UI Layer Action
    const regPage = new RegistrationPage(driver);
    await regPage.navigate();
    await regPage.fillForm('John Doe', testEmail, 'Password123!');
    await regPage.submit();
    // 2. UI Layer Assertion
    const successMsg = await regPage.getSuccessMessage();
    expect(successMsg).to.equal('Registration Successful');
    // 3. Database Layer Assertion
    const dbUser = await UserDAO.getUserByEmail(testEmail);
    // If the user is null, the UI lied! The API call failed silently!
    expect(dbUser).to.not.be.null;
    // Verify the data was stored correctly
    expect(dbUser.full_name).to.equal('John Doe');
    expect(dbUser.status).to.equal('ACTIVE');
  });
});
```

## Conclusion

By installing `mysql2` and writing a simple `UserDAO`, we have eliminated the "Silent API Failure" problem. We are now verifying the entire stack, from the Chrome browser all the way down to the SQL hard drive.

But what if your application doesn't use MySQL? What if it uses the modern, highly-scalable, open-source giant: **PostgreSQL**?

In the next tutorial, we will swap out our MySQL driver for Postgres and learn how to validate JSONB columns!
