---
title: The King of Open Source: PostgreSQL Testing
date: 19-Jan-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [typescript, selenium, postgresql, jsonb, database-testing, enterprise-validation, sql]
category: Selenium TypeScript
categories: [Selenium TypeScript, Enterprise Validation]
excerpt: >-
  Master enterprise data validation with PostgreSQL. Learn how to implement connection pooling, build DAOs, and assert complex JSONB payloads directly from your Selenium TypeScript tests.
readTime: 4 min read
---

# The King of Open Source: PostgreSQL Testing

In the previous tutorial, we successfully connected our Selenium framework to a MySQL database to validate backend data insertions.

However, over the last decade, a different open-source database has taken the throne as the default choice for modern Enterprise architectures: **PostgreSQL**.

Why is Postgres so popular? Because it combines the strict schema enforcement of traditional SQL with the incredible flexibility of NoSQL by providing native support for **JSONB** columns. 

As an Enterprise Automation Architect, you must know how to test PostgreSQL.

---

## 1. Installing the PostgreSQL Driver

To connect to a Postgres database in Node.js, we use the incredibly robust `pg` library.

Install the driver and its TypeScript definitions:

```bash
npm install pg
npm install --save-dev @types/pg
```

---

## 2. Establishing the Postgres Connection

Just like we did with MySQL, we will create a dedicated Manager class to handle the connection pool securely using environment variables.

In your `.env` file:

```env
PG_HOST=localhost
PG_PORT=5432
PG_USER=qa_admin
PG_PASSWORD=super_secret
PG_DATABASE=enterprise_db
```

Create `PostgresManager.ts`:

```typescript
import { Pool } from 'pg';
import * as dotenv from 'dotenv';
dotenv.config();
export class PostgresManager {
  private static pool: Pool | null = null;
  static getPool(): Pool {
    if (!this.pool) {
      this.pool = new Pool({
        host: process.env.PG_HOST,
        port: Number(process.env.PG_PORT),
        user: process.env.PG_USER,
        password: process.env.PG_PASSWORD,
        database: process.env.PG_DATABASE,
        max: 20, // Max number of connections in the pool
        idleTimeoutMillis: 30000
      });
      console.log('[DB] PostgreSQL Connection Pool initialized.');
    }
    return this.pool;
  }
  static async closePool() {
    if (this.pool) {
      await this.pool.end();
      this.pool = null;
      console.log('[DB] PostgreSQL Connection Pool closed.');
    }
  }
}
```
*Note: Notice how we use a `Pool` instead of a single Connection. This is crucial for running Selenium tests in parallel without dropping database connections!*

---

## 3. Querying JSONB Data in Postgres

Here is where Postgres shines. Imagine your application has an `orders` table. Instead of having 50 different columns for every possible product attribute, the developers just threw all the metadata into a single `metadata` column of type `JSONB`.

We need to verify that when a user orders a "Red T-Shirt", the `metadata` column correctly updates the JSON payload.

Create `OrderDAO.ts`:

```typescript
import { PostgresManager } from '../utils/PostgresManager';
export class OrderDAO {
  static async getOrderMetadata(orderId: string) {
    const pool = PostgresManager.getPool();
    // Postgres uses $1, $2 instead of ? for parameterized queries
    const query = 'SELECT metadata FROM orders WHERE order_id = $1';
    const result = await pool.query(query, [orderId]);
    if (result.rows.length === 0) return null;
    // The pg driver automatically parses the JSONB into a JS object!
    return result.rows[0].metadata; 
  }
}
```

---

## 4. The E2E Assertion

Finally, we tie it all together in our test file. We use Selenium to place the order, grab the Order ID from the UI, and then use our Postgres DAO to assert the JSON payload deep in the database.

```typescript
import { Builder, WebDriver } from 'selenium-webdriver';
import { expect } from 'chai';
import { CheckoutPage } from '../pages/CheckoutPage';
import { OrderDAO } from '../db/OrderDAO';
import { PostgresManager } from '../utils/PostgresManager';
describe('PostgreSQL Checkout Validation', function () {
  let driver: WebDriver;
  let createdOrderId: string;
  before(async function () {
    driver = await new Builder().forBrowser('chrome').build();
  });
  after(async function () {
    await driver.quit();
    await PostgresManager.closePool();
  });
  it('should correctly insert JSONB metadata on checkout', async function () {
    const checkoutPage = new CheckoutPage(driver);
    await checkoutPage.navigate();
    // 1. UI Layer Action
    await checkoutPage.selectColor('Red');
    await checkoutPage.selectSize('Large');
    await checkoutPage.submitOrder();
    // 2. UI Layer Extraction
    createdOrderId = await checkoutPage.getOrderId();
    expect(createdOrderId).to.be.a('string').and.not.empty;
    // 3. Database Layer Assertion
    const metadata = await OrderDAO.getOrderMetadata(createdOrderId);
    expect(metadata).to.not.be.null;
    // Validating the deeply nested JSONB payload!
    expect(metadata.itemDetails.color).to.equal('Red');
    expect(metadata.itemDetails.size).to.equal('Large');
    expect(metadata.warehouse.shipped).to.be.false;
  });
});
```

## Conclusion

By leveraging the `pg` driver and connection pooling, we can safely execute highly complex, data-driven validations against a Postgres architecture. Validating JSONB columns is a critical skill for any QA Engineer working in modern microservice environments.

However, if your entire backend consists *only* of JSON documents, you probably aren't using Postgres. You are probably using a NoSQL database.

In our next tutorial, we will explore the undisputed king of NoSQL: **MongoDB**!
