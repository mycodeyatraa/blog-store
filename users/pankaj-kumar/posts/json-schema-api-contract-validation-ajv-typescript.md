---
title: Guarantee Backend Contracts: JSON Schema Validation in TypeScript
date: 12-Dec-2024
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [typescript, selenium, api-testing, ajv, contract-testing, json-schema]
category: Selenium TypeScript
categories: [Selenium TypeScript, API Testing Integration]
excerpt: >-
  Prevent breaking changes from the backend by implementing strict JSON schema contract validation in your TypeScript API tests using ajv.
readTime: 4 min read
---

# Guarantee Backend Contracts: JSON Schema Validation in TypeScript

One of the most insidious bugs in software development occurs when the backend developers unexpectedly change an API response structure. They might rename `username` to `user_name`, or accidentally change an `id` field from an `integer` to a `string`. 

Your `200 OK` status assertions will pass, but your frontend UI will crash because the data format is wrong.

This is where **Contract Testing** comes in. By using JSON schemas, we can validate the exact structure and data types of an API response.

In this tutorial, we will use the `ajv` library to implement strict contract testing!

---

## 1. What is AJV?

`ajv` (Another JSON Schema Validator) is the fastest JSON schema validator for Node.js. It allows you to define a blueprint (a schema) of what your JSON response *should* look like, and it automatically cross-checks incoming API data against that blueprint.

## 2. Installing AJV

If you haven't already, install `ajv` in your project:

```bash
> npm install ajv
```

---

## 3. Writing the Contract Validation Script

Let's write a test that verifies the `/users/1` endpoint from `jsonplaceholder` exactly matches our expected contract.

Create `tests/api_schema.test.ts`:

```typescript
import { ApiUtils } from "./utils/ApiUtils";
import Ajv from "ajv";
const ajv = new Ajv(); // Create a new Ajv instance
// 1. Define the exact contract (schema) we expect the backend to honor
const userSchema = {
  type: "object",
  properties: {
    id: { type: "integer" },
    name: { type: "string" },
    username: { type: "string" },
    email: { type: "string" },
  },
  required: ["id", "name", "username", "email"],
  additionalProperties: true // Allow other fields to exist, but enforce the required ones
};
// 2. Compile the schema
const validate = ajv.compile(userSchema);
describe("Phase 6 - API Testing: Contract Validation", () => {
  it("Should validate the API response against a strict JSON Schema", async () => {
    // 3. Fetch data from the API
    const response = await ApiUtils.get("/users/1");
    expect(response.status).toBe(200);
    const payload = response.data;
    // 4. Validate the payload against the compiled schema!
    const isValid = validate(payload);
    if (!isValid) {
      console.error("Schema Validation Errors:", validate.errors);
    }
    // 5. The test will fail if the backend breaks the contract!
    expect(isValid).toBe(true);
    console.log("JSON Schema validation passed successfully! Contract honored.");
  });
});
```

### Breaking it down:
* **The Schema Definition:** We explicitly state that the response must be an `object` containing an `id` (integer), and `name`, `username`, and `email` (strings). 
* **`required` Array:** If the backend accidentally deletes the `email` field from the response, `ajv` will throw an error immediately, failing the test.
* **`validate()`:** This function compares our live API data (`payload`) against the compiled schema and returns `true` or `false`.

---

## 4. Test Execution Output

Run the test:

```bash
> ENV_NAME=qa jest tests/api_schema.test.ts
```

Output:

```text
 PASS  tests/api_schema.test.ts
  Phase 6 - API Testing: Contract Validation
    √ Should validate the API response against a strict JSON Schema (134 ms)
  console.log
    [API] Sending GET request to: https://jsonplaceholder.typicode.com/users/1
    [API] Received Status: 200
    JSON Schema validation passed successfully! Contract honored.
```

## Conclusion

Contract Testing provides an incredible safety net. Instead of writing dozens of manual assertions (e.g., `expect(typeof data.id).toBe("number")`), you define a single JSON schema that validates the entire payload structure instantaneously.

In our next and final set of tutorials, we will explore **API Framework Design** and how to architect enterprise-ready solutions by utilizing **POJOs (Plain Old JavaScript Objects)** to serialize and deserialize our data automatically!
