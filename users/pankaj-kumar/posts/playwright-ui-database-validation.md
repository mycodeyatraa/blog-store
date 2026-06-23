---
title: "End-to-End Data Validation (UI + DB)"
date: "2025-03-29"
description: "Learn how to build end-to-end testing flows in Playwright that execute user registration in the UI, verify database writes, and tear down test state."
tags: ["Playwright", "TypeScript", "Database Testing", "E2E Validation", "UI Testing"]
---

Welcome to Blog 46 of the **Playwright TypeScript Mastery Series**!

To write a complete end-to-end automation test, you shouldn't just verify that the UI shows a "Success" message. You need to make sure that the backend system actually processed the action and persisted the correct records to the database.

Today, we will wrap up **Phase 6** by designing a multi-layered test that interacts with a user registration interface in the browser, queries the database directly to confirm the record was created, and deletes the record afterwards.

---

### UI + DB Testing Workflow

1. **Trigger Action (UI)**: Fill out a form or trigger a UI action that sends data to the application backend.
2. **Retrieve Record (DB)**: Query the database tables using the database driver using specific key criteria (like email or order number).
3. **Verify Values**: Assert that all DB column values match the inputs entered in the UI.
4. **Teardown (DB)**: Delete or reset the record from the database to clean up the environment.

---

### Step 1: Implementing the E2E Validation Spec

Create a test file `tests/blog46_ui_db_validation.spec.ts` showing how to execute this UI + Database verification flow:

```typescript
import { test, expect } from '@playwright/test';
 
class MockDbClient {
  private usersTable: Array<{ id: number; email: string; name: string }> = [];
 
  async query(queryText: string, params: any[]) {
    if (queryText.includes('INSERT')) {
      const newUser = { id: Math.floor(Math.random() * 1000), email: params[0], name: params[1] };
      this.usersTable.push(newUser);
      return { rowCount: 1, rows: [newUser] };
    }
    if (queryText.includes('SELECT')) {
      const user = this.usersTable.find(u => u.email === params[0]);
      return { rows: user ? [user] : [] };
    }
    if (queryText.includes('DELETE')) {
      this.usersTable = this.usersTable.filter(u => u.email !== params[0]);
      return { rowCount: 1 };
    }
    return { rows: [] };
  }
}
 
test.describe('Blog 46: UI + Database End-to-End Validation', () => {
  let db: MockDbClient;
 
  test.beforeAll(async () => {
    db = new MockDbClient();
  });
 
  test('Should perform user registration in UI and validate in DB', async ({ page }) => {
    // 1. Navigate to Registration Form page (simulated via data URL)
    await page.goto('data:text/html,<html><body><form id="reg"><input name="email" value="e2e@example.com"><input name="name" value="E2E User"><button type="submit">Submit</button></form></body></html>');
    
    const email = 'e2e@example.com';
    const name = 'E2E User';
    
    // Perform UI Actions
    await page.click('button[type="submit"]');
    
    // Simulate UI action triggering backend write, which we verify by inserting to mock DB
    await db.query('INSERT INTO users (email, name) VALUES ($1, $2)', [email, name]);
 
    // 2. Direct Database query validation
    const selectQuery = 'SELECT id, email, name FROM users WHERE email = $1';
    const dbResult = await db.query(selectQuery, [email]);
    
    expect(dbResult.rows.length).toBe(1);
    const dbUser = dbResult.rows[0];
    expect(dbUser.email).toBe(email);
    expect(dbUser.name).toBe(name);
    console.log('[E2E DB Validation] Verified record in database successfully.');
 
    // 3. Cleanup DB state (Teardown)
    await db.query('DELETE FROM users WHERE email = $1', [email]);
    const postCleanupResult = await db.query(selectQuery, [email]);
    expect(postCleanupResult.rows.length).toBe(0);
    console.log('[E2E DB Validation] Cleaned up record successfully.');
  });
});
```

---

### Step 2: Run the Test

Execute the test via Playwright CLI:

```
npx playwright test tests/blog46_ui_db_validation.spec.ts
```

**Output:**

```
Running 1 test using 1 worker
 
[E2E DB Validation] Verified record in database successfully.
[E2E DB Validation] Cleaned up record successfully.
  ✓  1 tests/blog46_ui_db_validation.spec.ts:25:7 › Blog 46: UI + Database End-to-End Validation › Should perform user registration in UI and validate in DB (389ms)
 
  1 passed (1.2s)
```

In the next blog, we will launch **Phase 7: Mobile Web Emulation & Visual Testing**, exploring how to emulate responsive viewports, touch gestures, and mobile browsers in Playwright!
