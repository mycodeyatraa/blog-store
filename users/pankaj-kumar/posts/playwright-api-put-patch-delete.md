---
title: "Validating PUT, PATCH, and DELETE APIs"
date: "2025-03-21"
description: "Learn how to perform HTTP updates and resource deletions using Playwright's PUT, PATCH, and DELETE request configurations, and how to write assertions against success criteria."
tags: ["Playwright", "TypeScript", "API Testing", "Validation"]
---

Welcome to Blog 38 of the **Playwright TypeScript Mastery Series**!

To build comprehensive end-to-end API suites, you must handle resource lifecycles. This means not only fetching and creating data (GET and POST), but also updating and removing resources.

Today, we will learn how to send PUT, PATCH, and DELETE requests in Playwright, understand the differences in their application states, and write robust assertions for updates and deletions.

---

### Understanding PUT vs. PATCH vs. DELETE

- **PUT (Full Update)**: Replaces the target resource entirely with the new payload. If any fields are omitted from the payload, the server typically clears them or sets them to default values.
- **PATCH (Partial Update)**: Modifies only the specific fields provided in the payload, leaving all other existing properties untouched.
- **DELETE (Resource Removal)**: Permanently removes the target resource from the server.

---

### Step 1: Implementing the Test Spec

Create a test file `tests/blog38_api_put_patch_delete.spec.ts` to implement our update and deletion validation flow:

```typescript
import { test, expect } from '@playwright/test';
 
test.describe('Blog 38: Validating PUT, PATCH, and DELETE APIs in Playwright', () => {
 
  test('PUT Request - Full Resource Update', async ({ request }) => {
    const updatedPayload = {
      id: 1,
      title: 'Updated Post Title using PUT',
      body: 'This body replaces the previous body entirely.',
      userId: 1
    };
 
    // Send the PUT request with the full payload
    const response = await request.put('https://jsonplaceholder.typicode.com/posts/1', {
      data: updatedPayload
    });
 
    // Validate the response status is 200 OK
    expect(response.status()).toBe(200);
    expect(response.ok()).toBeTruthy();
 
    const body = await response.json();
    expect(body.title).toBe(updatedPayload.title);
    expect(body.body).toBe(updatedPayload.body);
    
    console.log('PUT Test Passed! Full resource updated successfully.');
  });
 
  test('PATCH Request - Partial Resource Update', async ({ request }) => {
    const partialPayload = {
      title: 'Updated Title via PATCH'
    };
 
    // Send the PATCH request with only the modified fields
    const response = await request.patch('https://jsonplaceholder.typicode.com/posts/1', {
      data: partialPayload
    });
 
    expect(response.status()).toBe(200);
    expect(response.ok()).toBeTruthy();
 
    const body = await response.json();
    expect(body.title).toBe(partialPayload.title);
    // Other properties should remain unchanged in the response
    expect(body.body).toBeDefined(); 
    
    console.log('PATCH Test Passed! Partial resource updated successfully.');
  });
 
  test('DELETE Request - Resource Removal', async ({ request }) => {
    // Send the DELETE request targeting a specific resource ID
    const response = await request.delete('https://jsonplaceholder.typicode.com/posts/1');
 
    // Verify response status
    // Note: JSONPlaceholder returns 200 OK.
    // In many production APIs, a successful DELETE will return 204 No Content instead.
    expect(response.status()).toBe(200);
    expect(response.ok()).toBeTruthy();
 
    console.log('DELETE Test Passed! Resource deleted successfully.');
  });
 
});
```

---

### Status Codes in Deletions: 200 vs 204

When validating a DELETE request, status expectations depend on the API design:
- **`200 OK`**: Typically returned if the response body contains a confirmation message or the deleted object representation.
- **`204 No Content`**: Typically returned if the deletion was successful and there is no response body to return (which is a standard RESTful design practice).

Playwright's `response.status()` allows you to assert on either outcome cleanly.

---

### Step 2: Run the Test

Execute the test via Playwright CLI:

```
npx playwright test tests/blog38_api_put_patch_delete.spec.ts
```

**Output:**

```
Running 3 tests using 1 worker
 
PUT Test Passed! Full resource updated successfully.
  ✓  1 tests/blog38_api_put_patch_delete.spec.ts:5:7 › Blog 38: Validating PUT, PATCH, and DELETE APIs in Playwright › PUT Request - Full Resource Update (727ms)
PATCH Test Passed! Partial resource updated successfully.
  ✓  2 tests/blog38_api_put_patch_delete.spec.ts:27:7 › Blog 38: Validating PUT, PATCH, and DELETE APIs in Playwright › PATCH Request - Partial Resource Update (637ms)
DELETE Test Passed! Resource deleted successfully.
  ✓  3 tests/blog38_api_put_patch_delete.spec.ts:47:7 › Blog 38: Validating PUT, PATCH, and DELETE APIs in Playwright › DELETE Request - Resource Removal (232ms)
 
  3 passed (2.7s)
```

In the next blog, we will cover **Authentication in API Testing** and learn how to manage cookies, tokens, and authorization headers in Playwright!
