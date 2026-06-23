---
title: "Manipulating LocalStorage & SessionStorage"
date: "2025-04-06"
description: "Learn how to read, write, and clear LocalStorage and SessionStorage values in Playwright TypeScript to test state-dependent application behavior."
tags: ["Playwright", "TypeScript", "LocalStorage", "SessionStorage"]
---

Welcome to Blog 54 of the **Playwright TypeScript Mastery Series**!

Many modern web applications rely heavily on client-side storage systems like `window.localStorage` and `window.sessionStorage` to persist themes, preferences, user data, or auth tokens. During automation testing, you often need to manipulate this storage to:
1. Seed the application state before performing checks.
2. Verify that your application correctly handles invalid or missing storage entries.
3. Clean up credentials and stored states between test cases.

Today, we will learn how to read, write, and clear storage objects in Playwright.

---

### Key Storage Restrictions

When dealing with browser storage:
- Storage is tied directly to the **origin** (domain, protocol, and port).
- You cannot read or write to `localStorage` or `sessionStorage` on a `data:` URI or `about:blank` page in some browsers due to security boundaries.
- You must navigate to a valid web page of the target origin *before* trying to evaluate storage operations.

---

### Step 1: Implementing the Storage Spec

Create a test file `tests/blog54_storage.spec.ts`:

```typescript
import { test, expect } from '@playwright/test';

test.describe('Blog 54: Manipulating LocalStorage and SessionStorage', () => {

  test('Read, write and clear storage', async ({ page }) => {
    // Navigate to a valid domain so storage is not blocked by browser security restrictions
    await page.goto('https://example.com');

    // 1. Set values in LocalStorage & SessionStorage
    await page.evaluate(() => {
      localStorage.setItem('userTheme', 'dark-mode');
      sessionStorage.setItem('sessionToken', 'temp_xyz123');
    });

    // 2. Read values and assert
    const theme = await page.evaluate(() => localStorage.getItem('userTheme'));
    const token = await page.evaluate(() => sessionStorage.getItem('sessionToken'));

    expect(theme).toBe('dark-mode');
    expect(token).toBe('temp_xyz123');
    console.log(`[Storage] Read LocalStorage: ${theme}, SessionStorage: ${token}`);

    // 3. Clear LocalStorage and verify
    await page.evaluate(() => localStorage.clear());
    const clearedTheme = await page.evaluate(() => localStorage.getItem('userTheme'));
    expect(clearedTheme).toBeNull();
  });

});
```

---

### Step 2: Running the Test

Execute the test via Playwright:

```
npx playwright test tests/blog54_storage.spec.ts
```

**Output:**

```
Running 1 test using 1 worker

[Storage] Read LocalStorage: dark-mode, SessionStorage: temp_xyz123
  ✓  1 tests/blog54_storage.spec.ts:5:7 › Blog 54: Manipulating LocalStorage and SessionStorage › Read, write and clear storage (3.0s)

  1 passed (4.3s)
```

In the next blog, we will take a step further into client state manipulation and learn how to **manage and set Cookies** in Playwright!
