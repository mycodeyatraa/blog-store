---
title: Mocking State: Mocking Component Props & State in Playwright CT
date: 09-Feb-2026
lastUpdated: 09-Feb-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: ["playwright", "typescript", "components", "mocking", "state"]
category: Component Testing
categories: ["Playwright TypeScript", "Component Testing", "UI Automation"]
excerpt: >-
  Master data boundary testing. Learn how to mock props, component state, context providers, and network requests inside Playwright CT.
readTime: 4 min read
---

# Mocking State: Mocking Component Props & State in Playwright CT

To test edges and error cases, we must feed custom props and mock application state (Redux, Contexts) into our mounted components.

---

## 1. Mocking Props and Context

Below is an execution sample mocking state:

```typescript
import { test, expect } from '@playwright/experimental-ct-react';
import ShoppingCart from '../src/components/ShoppingCart';
test('should display cart total matching custom state', async ({ mount }) => {
  const component = await mount(
    <ShoppingCart 
      initialItems={[{ id: 1, name: 'Book', price: 10 }]} 
    />
  );
  await expect(component.locator('.total')).toHaveText('$10.00');
});
```

This guarantees tests stay robust against backend network failures.
