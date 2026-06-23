---
title: "Database Mocking & Seeding"
date: "2025-03-28"
description: "Learn how to seed and mock databases with custom test datasets before executing tests, and set up automatic hooks for schema setup and data teardown."
tags: ["Playwright", "TypeScript", "Database Testing", "Data Seeding", "Teardown"]
---

Welcome to Blog 45 of the **Playwright TypeScript Mastery Series**!

When running integration and E2E tests, you need consistent data. If a test relies on checking a user profile, that user profile record must exist. If you rely on a manual setup, another run might modify or delete the record, resulting in a flaky test.

To resolve this, we use **Database Seeding & Teardown**. Seeding is the process of prepopulating database tables with a known set of test data before tests start. Teardown ensures that after the test run finishes, this test data is removed (truncated) so we leave the database in its original clean state.

---

### Seeding vs. Hardcoding Test Data

- **Hardcoding**: Relying on records that exist permanently in the database (e.g. `user_id = 12`). If this user gets updated or deleted, all tests fail.
- **Seeding**: Creating a fresh user record at the start of the test run, utilizing that record, and then deleting it at the end of the run. This guarantees a predictable environment.

---

### Step 1: Implementing the Seeding Spec

Create a test file `tests/blog45_db_seeding.spec.ts` showing how to use `beforeAll` and `afterAll` hooks to manage data lifecycles:

```typescript
import { test, expect } from '@playwright/test';

// Define simulated DB client with seed/teardown utility methods
class DatabaseManager {
  private usersTable: Array<{ id: number; email: string; name: string }> = [];

  async seedData() {
    console.log('[DB Manager] Seeding mock user database records...');
    this.usersTable = [
      { id: 101, email: 'user101@example.com', name: 'Alice Smith' },
      { id: 102, email: 'user102@example.com', name: 'Bob Johnson' }
    ];
  }

  async clearData() {
    console.log('[DB Manager] Cleaning up database (truncating tables)...');
    this.usersTable = [];
  }

  async getUser(id: number) {
    return this.usersTable.find(u => u.id === id);
  }
}

test.describe('Blog 45: Database Mocking & Seeding in Playwright', () => {
  let db: DatabaseManager;

  // Run before all tests to seed database
  test.beforeAll(async () => {
    db = new DatabaseManager();
    await db.seedData();
  });

  // Run after all tests to truncate database tables
  test.afterAll(async () => {
    await db.clearData();
  });

  test('Verify first seeded user can be fetched', async () => {
    const user = await db.getUser(101);
    expect(user).toBeDefined();
    expect(user?.email).toBe('user101@example.com');
    expect(user?.name).toBe('Alice Smith');
    
    console.log('[DB Seeding] Alice Smith verified successfully.');
  });

  test('Verify second seeded user can be fetched', async () => {
    const user = await db.getUser(102);
    expect(user).toBeDefined();
    expect(user?.email).toBe('user102@example.com');
    expect(user?.name).toBe('Bob Johnson');
    
    console.log('[DB Seeding] Bob Johnson verified successfully.');
  });

});
```

---

### Step 2: Run the Test

Execute the test via Playwright CLI:

```
npx playwright test tests/blog45_db_seeding.spec.ts
```

**Output:**

```
Running 2 tests using 1 worker

[DB Manager] Seeding mock user database records...
[DB Seeding] Alice Smith verified successfully.
  ✓  1 tests/blog45_db_seeding.spec.ts:39:7 › Blog 45: Database Mocking & Seeding in Playwright › Verify first seeded user can be fetched (10ms)
[DB Seeding] Bob Johnson verified successfully.
[DB Manager] Cleaning up database (truncating tables)...
  ✓  2 tests/blog45_db_seeding.spec.ts:48:7 › Blog 45: Database Mocking & Seeding in Playwright › Verify second seeded user can be fetched (8ms)

  2 passed (1.2s)
```

In the next blog, we will wrap up **Phase 6** by performing complete **End-to-End Data Validation** verifying changes across the UI, API, and Database simultaneously!
