---
title: "Contract Testing (JSON Schema Validation)"
date: "2025-03-24"
description: "Learn how to perform API Contract Testing in Playwright by validating JSON response schemas using the Ajv validator package."
tags: ["Playwright", "TypeScript", "API Testing", "Contract Testing"]
---

Welcome to Blog 41 of the **Playwright TypeScript Mastery Series**!

In microservices architectures, teams build services independently. If a backend team changes a response payload structure (e.g. renaming a field or changing an integer to a string), client applications can break silently. 

To prevent this, we write **Contract Tests**. Contract testing validates that the structure, keys, and data types of an API response conform strictly to a predefined schema.

Today, we will learn how to write contract tests in Playwright using **Ajv** (the standard JSON Schema validator for Node.js).

---

### Installing Ajv (Schema Validator)

To validate response payloads against JSON schemas, install the `ajv` package in your Playwright framework:

```
npm install ajv --save-dev
```

---

### Step 1: Implementing the Test Spec

Create a test file `tests/blog41_contract_testing.spec.ts` to implement our JSON Schema validation:

```typescript
import { test, expect } from '@playwright/test';
import Ajv from 'ajv';

// Instantiate Ajv schema validator
const ajv = new Ajv();

// Define our expected API response contract (JSON Schema)
const postSchema = {
  type: 'object',
  properties: {
    userId: { type: 'integer' },
    id: { type: 'integer' },
    title: { type: 'string' },
    body: { type: 'string' }
  },
  required: ['userId', 'id', 'title', 'body'],
  additionalProperties: false // Enforces strict schema with no unexpected fields
};

test.describe('Blog 41: Contract Testing in Playwright', () => {

  test('Validate GET API response against JSON Schema', async ({ request }) => {
    // 1. Fetch data from the API endpoint
    const response = await request.get('https://jsonplaceholder.typicode.com/posts/1');
    expect(response.status()).toBe(200);

    const responseBody = await response.json();

    // 2. Compile schema and validate response body
    const validate = ajv.compile(postSchema);
    const valid = validate(responseBody);

    // 3. Log errors if validation fails
    if (!valid) {
      console.error('AJV Schema Validation Errors:', validate.errors);
    }

    // 4. Assert response matches the contract
    expect(valid).toBeTruthy();
    console.log('Contract validation passed! Response conforms strictly to JSON Schema.');
  });

});
```

---

### Deciphering the Schema

In our `postSchema` definition:
- **`type: 'object'`**: Asserts that the response is a JSON object.
- **`properties`**: Defines each key and its required data type (e.g. `userId` must be an `integer`, `title` must be a `string`).
- **`required`**: Lists keys that *must* be present in the response. If any listed key is missing, validation fails.
- **`additionalProperties: false`**: Ensures that the API response does not contain any extra, undocumented fields. This is critical for strict contract enforcement.

---

### Step 2: Run the Test

Execute the test via Playwright CLI:

```
npx playwright test tests/blog41_contract_testing.spec.ts
```

**Output:**

```
Running 1 test using 1 worker

Contract validation passed! Response conforms strictly to JSON Schema.
  ✓  1 tests/blog41_contract_testing.spec.ts:22:7 › Blog 41: Contract Testing in Playwright › Validate GET API response against JSON Schema (367ms)

  1 passed (4.9s)
```

In the next blog, we will wrap up **Phase 5** by designing a full **API Automation Framework** in Playwright, complete with service endpoints, request helpers, and custom configurations!
