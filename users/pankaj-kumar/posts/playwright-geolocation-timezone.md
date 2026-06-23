---
title: "Geolocation & Timezone Emulation"
date: "2025-04-01"
description: "Learn how to emulate different geographical locations and system timezones in Playwright, spoofing coordinates and automatically bypassing permission prompts."
tags: ["Playwright", "TypeScript", "Mobile Emulation", "Geolocation", "Timezone"]
---

Welcome to Blog 49 of the **Playwright TypeScript Mastery Series**!

Many web applications tailor content, currency, date formatting, and functionality based on a user's location. If your app behaves differently for a visitor from New York than it does for a visitor from Tokyo, you need to verify location-based behavior automatically.

Today, we will learn how to configure timezone IDs, geolocation coordinates, and context permissions so you can spoof location settings in Playwright.

---

### Overriding Geo and Timezone Settings

In standard web browsers, geolocating a user requests permission via a pop-up prompt. If you don't grant permission, the application cannot get GPS coordinates.

Furthermore, HTML5 Geolocation API requires a **Secure Context** (HTTPS or localhost) to execute. On insecure connections, browsers completely disable the geolocation object on `navigator`.

Playwright overrides this interaction by:
1. **permissions**: Pre-approving the `'geolocation'` permission in the browser context so the pop-up does not display.
2. **geolocation**: Spoofing coordinates (latitude and longitude) that the browser's Geolocation API returns.
3. **timezoneId**: Overriding the internal JavaScript timezone engine (e.g. `Intl.DateTimeFormat().resolvedOptions().timeZone`).

---

### Step 1: Implementing the Geolocation Spec

Create a test file `tests/blog49_geolocation.spec.ts` demonstrating timezone and location overrides:

```typescript
import { test, expect } from '@playwright/test';
 
test.describe('Blog 49: Geolocation and Timezone Emulation in Playwright', () => {
  // Emulate Tokyo, Japan timezone and geolocation with permission granted
  test.use({
    geolocation: { latitude: 35.6762, longitude: 139.6503 },
    permissions: ['geolocation'],
    timezoneId: 'Asia/Tokyo'
  });
 
  test('Verify timezone and geolocation emulation', async ({ page }) => {
    // Navigate to a secure context website first
    await page.goto('https://example.com');
    
    // Inject elements and query APIs in the secure page context
    await page.evaluate(() => {
      const tzEl = document.createElement('div');
      tzEl.id = 'tz-info';
      tzEl.innerText = Intl.DateTimeFormat().resolvedOptions().timeZone;
      document.body.appendChild(tzEl);
 
      const geoEl = document.createElement('div');
      geoEl.id = 'geo-info';
      document.body.appendChild(geoEl);
 
      navigator.geolocation.getCurrentPosition(pos => {
        geoEl.innerText = pos.coords.latitude.toFixed(4) + ',' + pos.coords.longitude.toFixed(4);
      }, err => {
        geoEl.innerText = 'Error: ' + err.message;
      });
    });
    
    // Validate Timezone resolves correctly
    const tzText = await page.textContent('#tz-info');
    expect(tzText).toBe('Asia/Tokyo');
    console.log('[Geo Emulation] Timezone verified:', tzText);
 
    // Wait for geolocation callback to resolve and check results
    await expect(page.locator('#geo-info')).toHaveText('35.6762,139.6503');
    const geoText = await page.textContent('#geo-info');
    console.log('[Geo Emulation] Geolocation verified:', geoText);
  });
 
});
```

---

### Step 2: Run the Test

Execute the test via Playwright CLI:

```
npx playwright test tests/blog49_geolocation.spec.ts
```

**Output:**

```
Running 1 test using 1 worker
 
[Geo Emulation] Timezone verified: Asia/Tokyo
[Geo Emulation] Geolocation verified: 35.6762,139.6503
  ✓  1 tests/blog49_geolocation.spec.ts:11:7 › Blog 49: Geolocation and Timezone Emulation in Playwright › Verify timezone and geolocation emulation (3.2s)
 
  1 passed (4.5s)
```

In the next blog, we will dive into **Screenshot Comparisons (Visual Regression Testing)** to automate pixel-level comparisons of web pages!
