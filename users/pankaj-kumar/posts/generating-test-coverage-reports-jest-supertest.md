---
title: Generating Test Coverage Reports
date: 04-Mar-2025
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [supertest, jest, coverage, istanbul, reporting]
category: API Testing
categories: [API Testing, NodeJS, Automation, Analytics]
excerpt: >-
  Stop guessing if your APIs are fully tested. Learn how to generate beautiful tabular test coverage reports using Jest and SuperTest to catch missing edge cases.
readTime: 4 min read
---

When testing large Express applications, it's incredibly easy to accidentally forget to test an edge case or a specific error response. How do you objectively prove that your SuperTest suite is thoroughly evaluating your entire API?

Enter **Jest Coverage Reports**.

---

## 1. What is Test Coverage?

Test Coverage is a metric (usually expressed as a percentage) that tracks exactly how many lines of source code were executed during your test suite. 

Jest has built-in support for code coverage via the Istanbul framework. You simply add the `--coverage` flag!

## 2. Setting up the Target Code

To demonstrate this, I've created a tiny Express API with a deliberate edge case in `src/api.ts`:

```typescript
import express from 'express';
export const app = express();
app.use(express.json());
app.get('/status', (req, res) => {
    res.json({ up: true });
});
app.post('/calculate', (req, res) => {
    const { a, b } = req.body;
    if (a !== undefined && b !== undefined) {
        return res.json({ result: a + b });
    }
    // If our tests NEVER trigger a 400 error, Jest will flag this line!
    return res.status(400).json({ error: 'Missing parameters' });
});
```

## 3. Writing an Incomplete Test Suite

Now, let's write our SuperTest suite, but intentionally forget to write a test case for missing parameters.

```typescript
import request from 'supertest';
import { app } from '../src/api';
describe('Generating Test Coverage Reports', () => {
    it('1. Test the status endpoint', async () => {
        const response = await request(app).get('/status');
        expect(response.status).toBe(200);
    });
    it('2. Test the calculate endpoint (Happy Path)', async () => {
        const response = await request(app)
            .post('/calculate')
            .send({ a: 10, b: 5 });
        expect(response.status).toBe(200);
        expect(response.body.result).toBe(15);
    });
    // We INTENTIONALLY forgot to test the 400 Bad Request error!
});
```

## 4. Execution Results

When we run `npx jest --coverage tests/coverage.test.ts`, Jest will automatically intercept all Node.js imports, instrument the V8 engine, and map out exactly which lines of our `src/api.ts` file were executed.

```bash
PASS tests/coverage.test.ts
  Generating Test Coverage Reports
    [PASS] 1. Test the status endpoint (57 ms)
    [PASS] 2. Test the calculate endpoint (Happy Path) (32 ms)
----------|---------|----------|---------|---------|-------------------
File      | % Stmts | % Branch | % Funcs | % Lines | Uncovered Line #s 
----------|---------|----------|---------|---------|-------------------
All files |      90 |       75 |     100 |      90 |                   
 api.ts   |      90 |       75 |     100 |      90 | 16                
----------|---------|----------|---------|---------|-------------------
Test Suites: 1 passed, 1 total
Tests:       2 passed, 2 total
```

Look at that brilliant tabular output! Jest successfully detected that we achieved **90% Line Coverage**, but it explicitly warns us that **Line 16** (our 400 Error response) was completely uncovered! 

This tells us exactly what test we need to write next.

## 5. Conclusion

Test coverage is a superpower that ensures your APIs don't contain hidden dead code or unhandled exceptions. 

We only have two tutorials left! Next up, we will dive into a critical continuous integration subject: **Configuring CI/CD for Jest/SuperTest (GitHub Actions)**!
