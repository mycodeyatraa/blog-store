---
title: "Network Interception and Mocking"
date: "2025-03-10"
description: "Stop relying on unstable backends. Learn how to intercept network requests and mock API responses directly in Playwright."
tags: ["Playwright", "TypeScript", "Network", "Mocking", "Interception"]
---

Welcome to Blog 26 of the **Playwright TypeScript Mastery Series**!

Have your UI tests ever failed because the backend server was down? Or because the database was missing test data? This is the most common cause of "flaky tests" in the industry.

Playwright offers a superpower to solve this: **Network Interception**. Playwright can sit between your browser and the internet, intercept outgoing API requests, and instantly return fake (mocked) responses before the request ever reaches the real backend!

### Mocking an API Response

Using `page.route()`, you can intercept requests based on URL patterns. Let's write a test that simulates a frontend request and mocks the backend response.

Create `tests/blog26_network_mocking.spec.ts`:

```typescript
import { test, expect } from '@playwright/test';

test.describe('Blog 26: Network Interception & Mocking', () => {

  test('Mocking an API Response', async ({ page }) => {
    // 1. Tell Playwright to intercept any request matching this URL pattern
    await page.route('**/api/v1/status', async (route) => {
      // 2. Instead of letting the request hit the real server,
      // we fulfill it locally with our own fake JSON response!
      await route.fulfill({
        status: 200,
        contentType: 'application/json',
        body: JSON.stringify({ message: 'Playwright Intercepted This!' })
      });
    });

    await page.goto('https://practice.mycodeyatra.com');

    // 3. Let's simulate the frontend making a fetch request to that API
    const responseJson = await page.evaluate(async () => {
      const res = await fetch('https://practice.mycodeyatra.com/api/v1/status');
      return res.json();
    });

    // 4. Verify the frontend received our mocked data!
    expect(responseJson.message).toBe('Playwright Intercepted This!');
    
    console.log('Successfully intercepted and mocked the network request!');
  });
});
```

### Aborting Network Requests

Mocking isn't just for returning fake data. You can also completely **abort** requests. This is extremely useful for speeding up test execution by blocking heavy images, analytics trackers, or third-party ad networks!

```typescript
test('Blocking Network Requests (e.g., Images)', async ({ page }) => {
  // Intercept any URL ending in .png, .jpg, or .jpeg
  await page.route('**/*.{png,jpg,jpeg}', (route) => {
    route.abort(); // Block the request immediately
  });

  await page.goto('https://practice.mycodeyatra.com');
  
  // The page loads significantly faster because it didn't download images!
  await expect(page.locator('h1').first()).toBeVisible();
});
```

### Execution Output

When you run `npx playwright test tests/blog26_network_mocking.spec.ts`:

```text
Running 2 tests using 1 worker

Successfully intercepted and mocked the network request!
  ✓  1 tests\blog26_network_mocking.spec.ts:5:7 › Blog 26: Network Interception & Mocking › Mocking an API Response (3.5s)
Successfully loaded page while blocking all images!
  ✓  2 tests\blog26_network_mocking.spec.ts:32:7 › Blog 26: Network Interception & Mocking › Blocking Network Requests (e.g., Images) (619ms)

  2 passed (5.5s)
```

### Conclusion

By intercepting and mocking API responses, you completely decouple your UI tests from backend instability. You can test edge cases (like 500 Server Errors) simply by telling `route.fulfill({ status: 500 })`, without having to break your actual server!

In **Blog 27**, we will explore how to test **Multiple Browser Contexts** concurrently!
