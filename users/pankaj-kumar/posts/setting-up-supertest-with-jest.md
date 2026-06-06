---
title: Setting up Supertest with Jest/Vitest
date: 20-Feb-2025
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [supertest, nodejs, jest, typescript, api-testing]
category: API Testing
categories: [API Testing, NodeJS, Automation, TypeScript]
excerpt: >-
  Learn how to build a robust NodeJS API testing framework from scratch using SuperTest, Jest, and TypeScript, including powerful testing hooks.
readTime: 5 min read
---

In our previous post, we got a taste of SuperTest by firing a basic GET request. However, writing a single file isn't enough to build an enterprise-grade automation framework.

To truly unleash SuperTest, you must pair it with a test runner. The industry standard in the NodeJS ecosystem is **Jest** (or its faster, modern cousin **Vitest**). 

In this article, we will configure an enterprise-ready API testing framework using SuperTest, Jest, and TypeScript.

---

## 1. Initializing the Project

First, let's create a new NodeJS project and initialize a `package.json`:

```bash
mkdir my-api-tests
cd my-api-tests
npm init -y
```

## 2. Installing Dependencies

We need SuperTest for the HTTP calls, Jest for the test execution, and TypeScript for the type definitions.

```bash
npm install --save-dev supertest jest typescript ts-jest 
npm install --save-dev @types/supertest @types/jest @types/node
```

## 3. Configuring Jest and TypeScript

We installed `ts-jest` because Jest doesn't natively understand TypeScript out of the box. We need to create a `jest.config.js` to tell Jest to use the TypeScript preprocessor.

Create a file named `jest.config.js` at the root of your project:

```javascript
/** @type {import('ts-jest').JestConfigWithTsJest} */
module.exports = {
  preset: 'ts-jest',
  testEnvironment: 'node',
  testMatch: ['**/*.test.ts'],
};
```

Next, initialize a `tsconfig.json` so the TypeScript compiler knows how to handle our modern imports:

```json
{
  "compilerOptions": {
    "target": "es2016",
    "module": "commonjs",
    "esModuleInterop": true,
    "forceConsistentCasingInFileNames": true,
    "strict": true,
    "skipLibCheck": true,
    "types": ["jest", "node"]
  }
}
```

## 4. Writing a Test with Jest Hooks

Jest provides powerful setup and teardown hooks (`beforeAll`, `afterAll`, `beforeEach`, `afterEach`). These are incredibly useful in API testing for establishing database connections or fetching Authentication Tokens before tests run.

Create a new directory `tests` and add a file `setup.test.ts`:

```typescript
import request from 'supertest';
describe('Jest/SuperTest Setup', () => {
    // Define a dynamic API URL based on Environment Variables
    const API_URL = process.env.API_URL || 'http://localhost:8080';
    beforeAll(() => {
        // Setup code executes once before all tests in this block
        console.log(`Starting Test Suite against API: ${API_URL}`);
    });
    afterAll(() => {
        // Teardown code executes once after all tests have finished
        console.log('Cleaning up resources...');
    });
    it('should correctly hit the API', async () => {
        const response = await request(API_URL).get('/api/health');
        expect(response.status).toBe(200);
        expect(response.body).toBeDefined();
    });
});
```

## 5. Executing the Test

Finally, update your `package.json` to include a test script:

```json
  "scripts": {
    "test": "jest"
  }
```

Now, simply run your test framework using:
```bash
npm test
```

Jest will automatically locate your `.test.ts` files, compile them via `ts-jest`, and execute your SuperTest HTTP requests beautifully! In our next article, we will dive deeper into writing fully-fledged CRUD operations.
