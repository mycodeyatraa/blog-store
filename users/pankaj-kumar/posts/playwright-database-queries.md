---
title: "Executing Database Queries in Tests"
date: "2025-03-27"
description: "Learn how to execute parameterized SELECT and INSERT SQL queries in Playwright tests using async/await, validate row results, and avoid SQL injection security risks."
tags: ["Playwright", "TypeScript", "Database Testing", "SQL Queries", "pg"]
---

Welcome to Blog 44 of the **Playwright TypeScript Mastery Series**!

In the previous blog, we connected our automation suite to PostgreSQL. Today, we will learn how to run actual database queries (like `SELECT` and `INSERT`) dynamically from inside our tests. 

We will cover query parameters to protect against SQL Injection, mapping raw row datasets, and asserting database states inside Playwright test blocks.

---

### Security First: Parameterized Queries

When querying databases in tests, NEVER use string concatenation to embed test parameters:

```typescript
// BAD: Vulnerable to SQL injection!
const query = `SELECT * FROM users WHERE email = '${userEmail}'`;
```

Always use **parameterized queries (bind parameters)**. This ensures values are properly escaped and processed as literals by the DB engine, preventing vulnerabilities and format breaks:

```typescript
// GOOD: Safe from SQL injection!
const query = 'SELECT * FROM users WHERE email = $1';
const result = await client.query(query, [userEmail]);
```

---

### Step 1: Implementing the Queries Spec

Create a test file `tests/blog44_db_queries.spec.ts` showing how to execute and assert query result blocks:

```typescript
import { test, expect } from '@playwright/test';

// Define a helper demonstrating database query execution
class MockDbClient {
  async query(queryText: string, params?: any[]) {
    // Simulated DB query execution log
    console.log(`[Mock DB] Executing query: ${queryText}`);
    if (params) {
      console.log(`[Mock DB] Bind parameters:`, params);
    }
    
    if (queryText.includes('SELECT')) {
      return {
        rows: [
          { id: 1, email: 'user@example.com', name: 'John Doe', status: 'ACTIVE' }
        ]
      };
    }
    
    if (queryText.includes('INSERT')) {
      return { rowCount: 1 };
    }
    
    if (queryText.includes('DELETE')) {
      return { rowCount: 1 };
    }
    
    return { rows: [] };
  }
}

test.describe('Blog 44: Executing Database Queries in Playwright', () => {
  let db: MockDbClient;

  test.beforeEach(() => {
    db = new MockDbClient();
  });

  test('Execute SELECT query to verify database records', async () => {
    const userId = 1;
    // Parameterized query to avoid SQL Injection
    const sql = 'SELECT id, email, name, status FROM users WHERE id = $1';
    
    const result = await db.query(sql, [userId]);
    
    // Validate rows returned
    expect(result.rows.length).toBe(1);
    
    const user = result.rows[0];
    expect(user.id).toBe(1);
    expect(user.email).toBe('user@example.com');
    expect(user.status).toBe('ACTIVE');
    
    console.log('[DB Queries] SELECT query verification complete.');
  });

  test('Execute INSERT query to seed dynamic test user', async () => {
    const sql = 'INSERT INTO users (email, name, status) VALUES ($1, $2, $3)';
    const params = ['jane.doe@example.com', 'Jane Doe', 'ACTIVE'];
    
    const result = await db.query(sql, params);
    
    // Verify records created
    expect(result.rowCount).toBe(1);
    console.log('[DB Queries] INSERT query verification complete.');
  });

});
```

---

### Step 2: Run the Test

Execute the tests using the Playwright CLI:

```
npx playwright test tests/blog44_db_queries.spec.ts
```

**Output:**

```
Running 2 tests using 1 worker

[Mock DB] Executing query: SELECT id, email, name, status FROM users WHERE id = $1
[Mock DB] Bind parameters: [ 1 ]
[DB Queries] SELECT query verification complete.
  ✓  1 tests/blog44_db_queries.spec.ts:39:7 › Blog 44: Executing Database Queries in Playwright › Execute SELECT query to verify database records (18ms)
[Mock DB] Executing query: INSERT INTO users (email, name, status) VALUES ($1, $2, $3)
[Mock DB] Bind parameters: [ 'jane.doe@example.com', 'Jane Doe', 'ACTIVE' ]
[DB Queries] INSERT query verification complete.
  ✓  2 tests/blog44_db_queries.spec.ts:57:7 › Blog 44: Executing Database Queries in Playwright › Execute INSERT query to seed dynamic test user (6ms)

  2 passed (1.1s)
```

In the next blog, we will learn how to handle **Database Mocking and Seeding** to prepopulate tables before test executions!
