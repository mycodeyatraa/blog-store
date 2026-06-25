---
title: Full-Stack Enterprise Validation: Database Testing with Playwright
date: 26-Apr-2025
lastUpdated: 26-Apr-2025
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: ["playwright", "typescript", "database", "postgres", "sql", "full-stack"]
category: Enterprise Validation
categories: ["Enterprise Validation", "UI Automation", "Playwright", "TypeScript", "Database"]
excerpt: >-
  Elevate your test strategy by integrating PostgreSQL directly into your Playwright framework to seed data and assert backend states.
readTime: 5 min read
---

Enterprise end-to-end automation goes beyond just clicking buttons in a browser. Often, you need to query the database directly to verify that a backend transaction succeeded, or to seed the database with specific test data *before* your UI test executes.

Because Playwright is a standard Node.js application, it can integrate with absolutely any database system (PostgreSQL, MySQL, MongoDB, Redis) using standard NPM packages!

In this tutorial, we will explore how to natively connect Playwright to a PostgreSQL database using the `pg` library.

### 1. Installation

First, install the required PostgreSQL driver and its TypeScript definitions:

```bash
npm install pg
npm install -D @types/pg
```

### 2. Creating the Connection

Inside your Playwright test file, you instantiate a connection pool. A pool manages multiple connections dynamically and is highly recommended over single client connections for test stability.

**File:** `tests/database.spec.ts`

```typescript
import { test, expect } from '@playwright/test';
import { Pool } from 'pg';
 
// Create a connection pool for the database
const pool = new Pool({
  host: 'localhost',
  user: 'testuser',
  password: 'testpassword',
  database: 'testdb',
  port: 5432,
});
 
test.describe('Database Validation Tests', () => {
 
  // Ensure the connection pool is cleanly closed after the suite finishes
  test.afterAll(async () => {
    await pool.end();
  });
  
  // ... tests ...
});
```

### 3. Asserting Database State

Let's write a test that verifies an API or UI action actually resulted in a new row being created in the database.

```typescript
test('Verify user record exists in the database', async () => {
  const emailToTest = 'user@example.com';
  
  // 1. Query the database natively using parameterized queries to prevent SQL injection
  const result = await pool.query('SELECT id, name, status FROM users WHERE email = $1', [emailToTest]);
  
  // 2. Assert the database state using Playwright's expect()
  expect(result.rowCount).toBeGreaterThan(0);
  expect(result.rows[0].status).toBe('ACTIVE');
});
```

### 4. Database Seeding for UI Tests

A powerful pattern is to use the database to "seed" exact test conditions before launching the browser. This eliminates the need to rely on slow, brittle UI steps to set up the environment.

```typescript
test('Seed database data before UI execution', async ({ page }) => {
  const testUsername = 'ui_tester_99';
  
  // 1. Seed the database with a test user natively
  await pool.query('INSERT INTO users (name, email, status) VALUES ($1, $2, $3)', 
    [testUsername, 'tester99@example.com', 'ACTIVE']);
    
  // 2. Perform UI actions expecting the seeded data
  await page.goto('/login');
  await page.fill('#username', testUsername);
  await page.click('#login-btn');
  
  // Assert successful UI login
  await expect(page.locator('.welcome-message')).toBeVisible();
  
  // 3. Cleanup the database after the test
  await pool.query('DELETE FROM users WHERE name = $1', [testUsername]);
});
```

### Summary

Integrating database drivers directly into Playwright `spec.ts` files unlocks true full-stack automation. You can bypass the UI entirely to set up complex test states, drastically speeding up execution times and reducing flakiness!
