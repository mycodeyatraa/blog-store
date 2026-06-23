---
title: "Connecting to Databases (MySQL/PostgreSQL)"
date: "2025-03-26"
description: "Learn how to establish connections to MySQL and PostgreSQL databases within your Playwright TypeScript test automation framework, load credentials securely, and handle connection errors."
tags: ["Playwright", "TypeScript", "Database Testing", "PostgreSQL", "MySQL"]
---

Welcome to Blog 43 of the **Playwright TypeScript Mastery Series**!

To write comprehensive end-to-end (E2E) tests, you often need to verify that actions performed in the UI (like completing an order or updating a profile) successfully write or modify records in the database.

Today, we kick off **Phase 6: Database Integration & Testing**. We will learn how to configure database connection credentials securely, connect to PostgreSQL (or MySQL) databases using Node.js drivers, and manage connections gracefully.

---

### Why Integrate Databases in Playwright?

1. **State Verification**: Directly query the DB to verify that UI actions persisted the correct values.
2. **Test Data Seeding**: Insert fresh, valid test data records before tests start.
3. **Data Cleanup**: Clean up (teardown) created records after tests finish to keep environment data clean.
4. **Environment Independent**: Write queries to fetch active parameters (like feature toggles or API tokens) dynamically.

---

### Installing Database Drivers

To connect to database servers in your Node.js/Playwright project:

For **PostgreSQL**:

```
npm install pg @types/pg --save-dev
```

For **MySQL**:

```
npm install mysql2 --save-dev
```

---

### Step 1: Implementing the Connection Spec

Create a test file `tests/blog43_db_connection.spec.ts` to show how database configurations are loaded and connections are initiated:

```typescript
import { test, expect } from '@playwright/test';
import { Client } from 'pg';
 
test.describe('Blog 43: Connecting to PostgreSQL in Playwright', () => {
 
  test('Should configure database connection credentials', async () => {
    // 1. Define configuration options (best retrieved from process.env)
    const dbConfig = {
      host: process.env.DB_HOST || 'localhost',
      port: Number(process.env.DB_PORT) || 5432,
      database: process.env.DB_NAME || 'test_db',
      user: process.env.DB_USER || 'postgres',
      password: process.env.DB_PASSWORD || 'password123',
    };
 
    expect(dbConfig.host).toBeDefined();
    expect(dbConfig.port).toBe(5432);
    expect(dbConfig.database).toBe('test_db');
 
    console.log('[DB Integration] Database configuration loaded successfully.');
  });
 
  test('Attempt database connection (Graceful Failover)', async () => {
    const client = new Client({
      host: 'invalid-host-for-testing', // Intentionally wrong to demonstrate connection error handling
      port: 5432,
      database: 'test_db',
      user: 'postgres',
      password: 'password123',
      connectionTimeoutMillis: 1000 // Short timeout for rapid execution
    });
 
    try {
      await client.connect();
      console.log('[DB Integration] Connection established successfully.');
      await client.end();
    } catch (error) {
      console.log(`[DB Integration] Expected connection error caught: ${error.message}`);
      // Assert that connection failure was caught gracefully
      expect(error).toBeDefined();
      expect(error.message).toMatch(/timeout expired|getaddrinfo ENOTFOUND|ENOTFOUND/);
    }
  });
 
});
```

---

### Managing Client Lifecycles

When working with database connections in tests:
- **Connect Early**: Initiate connection prior to running queries.
- **Always Close Connections**: Use `client.end()` or connection pool releases to free resources. Leaving open connections will prevent your test runner from exiting!

---

### Step 2: Run the Test

Execute the test suite using the Playwright CLI:

```
npx playwright test tests/blog43_db_connection.spec.ts
```

**Output:**

```
Running 2 tests using 1 worker
 
[DB Integration] Database configuration loaded successfully.
  ✓  1 tests/blog43_db_connection.spec.ts:6:7 › Blog 43: Connecting to PostgreSQL in Playwright › Should configure database connection credentials (14ms)
[DB Integration] Expected connection error caught: timeout expired
  ✓  2 tests/blog43_db_connection.spec.ts:23:7 › Blog 43: Connecting to PostgreSQL in Playwright › Attempt database connection (Graceful Failover) (1.0s)
 
  2 passed (2.5s)
```

In the next blog, we will learn how to **Execute Database Queries** directly from our tests to insert, select, and delete data!
