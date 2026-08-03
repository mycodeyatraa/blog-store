---
title: Mocking Component Props & State in Playwright CT
date: 09-Feb-2026
lastUpdated: 09-Feb-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: ["playwright", "typescript", "component-testing", "mocking", "react"]
category: Component Testing
categories: ["Playwright TypeScript", "Component Testing", "Mocking"]
excerpt: >-
  Master state and prop injection in Playwright Component Testing! Validate complex UI states without backend dependencies.
readTime: 7 min read
---

# Mocking Component Props & State in Playwright CT

Testing edge-case UI states (e.g., error banners, loading skeletons, disabled buttons) is difficult when relying on live API responses. **Playwright CT** makes it effortless to pass custom props and stub internal state directly.

---

## 1. Mocking Complex Props in React Component Tests

```tsx
import { test, expect } from '@playwright/experimental-ct-react';
import { UserProfileCard } from './UserProfileCard';
 
test('should display danger badge for suspended user accounts', async ({ mount }) => {
  const mockUser = {
    id: 101,
    name: 'Pankaj Kumar',
    status: 'SUSPENDED',
    roles: ['ADMIN']
  };
 
  const component = await mount(<UserProfileCard user={mockUser} />);
 
  await expect(component.getByTestId('status-badge')).toHaveText('Account Suspended');
  await expect(component.getByTestId('status-badge')).toHaveClass(/badge-danger/);
});
```
