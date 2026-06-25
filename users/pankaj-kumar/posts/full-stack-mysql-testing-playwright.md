---
title: Database Testing with Playwright and MySQL
date: 27-Apr-2025
lastUpdated: 27-Apr-2025
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: ["playwright", "typescript", "database", "mysql", "sql", "full-stack"]
category: Enterprise Validation
categories: ["Enterprise Validation", "UI Automation", "Playwright", "TypeScript", "Database"]
excerpt: >-
  Master MySQL database integration in Playwright using mysql2 to natively query backend transactions and seed UI test data.
readTime: 5 min read
---

Following up on our PostgreSQL database tutorial, today we will tackle how to deeply integrate **MySQL** into your Playwright testing framework.

Connecting directly to your backend MySQL database is vital for validating that API transactions actually hit the physical storage, or for dynamically injecting session tokens before UI tests to bypass slow login screens.

### 1. Installation

Playwright is simply a Node.js process, which means we can utilize the standard, blazingly fast `mysql2` driver. It comes with built-in Promise support, which is critical for modern `async/await` automation.

```bash
npm install mysql2
```

### 2. Setting Up the Connection Pool

Instead of creating individual, slow database connections for every single test, we will create a global **Connection Pool**. A pool automatically manages a queue of connections and reuses them, which is essential when Playwright executes tests in parallel across multiple worker threads!

**File:** `tests/mysql.spec.ts`

```typescript
import { test, expect } from '@playwright/test';
import * as mysql from 'mysql2/promise';
 
// Define the MySQL connection pool
const pool = mysql.createPool({
  host: 'localhost',
  user: 'testuser',
  password: 'testpassword',
  database: 'testdb',
  waitForConnections: true,
  connectionLimit: 10,
  queueLimit: 0
});
 
test.describe('MySQL Database Integration', () => {
 
  // Ensure connections are closed after the test suite finishes to prevent hanging processes
  test.afterAll(async () => {
    await pool.end();
  });
  
  // ... tests go here ...
});
```

### 3. Asserting Backend State

A common enterprise pattern is triggering an action via an API or UI, and then ensuring the backend database was updated properly.

```typescript
  test('Verify API transaction reached MySQL database', async ({ request }) => {
    const orderId = 'ORD-789012';
    
    // 1. Simulate an API request to create an order
    const response = await request.post('/api/orders', { data: { id: orderId, amount: 250 } });
    expect(response.ok()).toBeTruthy();
 
    // 2. Query MySQL to verify the backend processed the transaction
    // Note: We use parameterized queries `?` to prevent SQL Injection
    const [rows]: any = await pool.query('SELECT status, amount FROM orders WHERE id = ?', [orderId]);
    
    // 3. Assert the state natively
    expect(rows.length).toBeGreaterThan(0);
    expect(rows[0].status).toBe('PROCESSING');
  });
```

### 4. Database Seeding & Teardown

Relying on the UI to reset test data is notoriously brittle. Instead, use native MySQL queries to clean up the environment in a fraction of a second.

```typescript
  test('Data Teardown via MySQL', async ({ page }) => {
    const sessionToken = 'session-xyz-123';
    
    // 1. Seed active session for UI test
    await pool.query('INSERT IGNORE INTO active_sessions (token, user_id) VALUES (?, ?)', [sessionToken, 999]);
    
    // 2. Perform UI actions utilizing the injected state
    await page.setExtraHTTPHeaders({ 'Authorization': `Bearer ${sessionToken}` });
    await page.goto('/dashboard');
    
    // 3. Clean up database state natively instead of relying on the UI
    await pool.query('DELETE FROM active_sessions WHERE token = ?', [sessionToken]);
  });
```

### Summary

Direct database connections elevate your Playwright suite from a mere frontend scraper into a robust full-stack validation tool! By leveraging `mysql2/promise`, you can natively manipulate application state and write highly deterministic, lightning-fast end-to-end tests.
