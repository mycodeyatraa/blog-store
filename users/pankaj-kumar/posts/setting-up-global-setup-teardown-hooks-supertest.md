---
title: Setting up Global Setup & Teardown Hooks
date: 02-Mar-2025
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [supertest, jest, hooks, beforeall, aftereach, framework]
category: API Testing
categories: [API Testing, NodeJS, Automation, Architecture]
excerpt: >-
  Architect scalable automation frameworks by mastering Jest lifecycle hooks. Learn how to manage state, databases, and Auth tokens efficiently across SuperTest suites.
readTime: 6 min read
---

As your API test suite grows from 10 tests to 1,000 tests, organizing setup and teardown logic becomes critical. You do not want to hardcode database connection logic or JWT generation inside every single `it()` block.

By leveraging **Jest's Lifecycle Hooks**, you can write modular, DRY (Don't Repeat Yourself) SuperTest suites that automatically manage external dependencies.

---

## 1. The Four Core Hooks

Jest provides four primary lifecycle methods that execute relative to your `describe()` block:

1. `beforeAll()`: Runs exactly once before the first test.
2. `beforeEach()`: Runs repeatedly before every single test.
3. `afterEach()`: Runs repeatedly after every single test.
4. `afterAll()`: Runs exactly once after the last test finishes.

Let's look at how we can implement these to manage state for our SuperTest executions:

```typescript
import request from 'supertest';
describe('Setting up Global Setup & Teardown Hooks', () => {
    const API_URL = 'http://localhost:8080';
    let dbConnectionId: string;
    // 1. beforeAll: Perfect for Database connections or generating global auth tokens.
    beforeAll(async () => {
        // Simulate an expensive operation that we only want to do ONCE
        dbConnectionId = 'mock_db_12345';
        console.log(`[Hooks] beforeAll: Connected to DB (${dbConnectionId})`);
    });
    // 2. beforeEach: Perfect for resetting state or clearing caches.
    beforeEach(() => {
        console.log('[Hooks] beforeEach: Resetting mock cache for the next test...');
    });
    it('1. Execute Test A', async () => {
        console.log('[Hooks] Executing Test A');
        const response = await request(API_URL).get('/api/health');
        expect(response.status).toBe(200);
        expect(dbConnectionId).toBe('mock_db_12345'); // Inherits state from beforeAll!
    });
    it('2. Execute Test B', async () => {
        console.log('[Hooks] Executing Test B');
        const response = await request(API_URL).get('/api/health');
        expect(response.status).toBe(200);
    });
    // 3. afterEach: Perfect for deleting specific artifacts created by a specific test.
    afterEach(() => {
        console.log('[Hooks] afterEach: Cleaning up test-specific data...');
    });
    // 4. afterAll: Perfect for gracefully closing database/socket connections.
    afterAll(() => {
        dbConnectionId = '';
        console.log('[Hooks] afterAll: Disconnected from DB. Teardown complete.');
    });
});
```

## 2. Global Setup Files

If you want a hook to run before the *entire suite* of files (not just a single `describe` block), you can configure Jest's `globalSetup` in your `jest.config.js`:

```javascript
module.exports = {
  preset: 'ts-jest',
  testEnvironment: 'node',
  globalSetup: './tests/setup.ts',
  globalTeardown: './tests/teardown.ts'
};
```
This is where you would traditionally spin up a Docker container holding your mock database before SuperTest ever fires its first HTTP request!

## 3. Execution Results

When we run `npx jest tests/hooks.test.ts`, observe the chronological order of the `console.log` statements in the terminal output. Notice how the `beforeEach` and `afterEach` hooks wrap *around* Test A and Test B individually:

```bash
  console.log
    [Hooks] beforeAll: Connected to DB (mock_db_12345)
  console.log
    [Hooks] beforeEach: Resetting mock cache for the next test...
  console.log
    [Hooks] Executing Test A
  console.log
    [Hooks] afterEach: Cleaning up test-specific data...
  console.log
    [Hooks] beforeEach: Resetting mock cache for the next test...
  console.log
    [Hooks] Executing Test B
  console.log
    [Hooks] afterEach: Cleaning up test-specific data...
  console.log
    [Hooks] afterAll: Disconnected from DB. Teardown complete.
PASS tests/hooks.test.ts
  Setting up Global Setup & Teardown Hooks
    [PASS] 1. Execute Test A (114 ms)
    [PASS] 2. Execute Test B (19 ms)
```

## 4. Conclusion

Mastering Hooks is the difference between a Junior QA Engineer and an Automation Architect. By abstracting boilerplate logic out of your test definitions, your SuperTest suites become infinitely more readable and robust.

In our next tutorial, we will learn how to mock third-party dependencies using **Mocking Axios / Fetch Requests**!
