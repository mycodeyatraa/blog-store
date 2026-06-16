---
title: Holding the Line: API Contract Validation with JSON Schema
date: 06-Dec-2024
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [typescript, selenium, api-testing, json-schema, ajv, contract-testing]
category: Selenium TypeScript
categories: [Selenium TypeScript, API Testing Integration]
excerpt: >-
  Prevent breaking backend changes from silently passing your tests. Learn how to use the Ajv library to enforce strict JSON Schema contracts on all API responses.
readTime: 4 min read
---

# Holding the Line: API Contract Validation with JSON Schema

Welcome to the **114th and FINAL tutorial** of the TypeScript Selenium Mastery series!

We have built a framework that can click buttons, read databases, and send dynamic REST requests. But what happens if the backend developers silently change the API response? What if `userId` is suddenly changed to `user_id`?

Your `ApiUtils.get()` request will still return `200 OK`, but your tests will fail down the line because the data structure changed!

To prevent this, we use **Contract Validation**. We define a strict "Schema" that the API must honor. If the shape of the data changes, our tests will instantly flag a breach of contract.

We will do this using **JSON Schema** and the `ajv` library!

---

## 1. Installing Ajv

`ajv` is the fastest JSON Schema validator for Node.js.

```bash
npm install ajv
```

---

## 2. Defining the Contract

Let's write a test that hits our `/users/1` endpoint. We will define a strict schema stating that the response **MUST** be an object containing an `id` (integer), `name` (string), `username` (string), and `email` (string).

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
  // If any of these fields go missing, the contract is breached!
  required: ["id", "name", "username", "email"],
  additionalProperties: true
};
// 2. Compile the schema
const validate = ajv.compile(userSchema);
describe("Phase 5 - API Testing: Contract Validation", () => {
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

---

## 3. Test Execution Output

```bash
> jest tests/api_schema.test.ts
```

Output:

```text
 PASS  tests/api_schema.test.ts
  Phase 5 - API Testing: Contract Validation
    √ Should validate the API response against a strict JSON Schema (128 ms)
  console.log
    JSON Schema validation passed successfully! Contract honored.
```

If the developers ever rename `email` to `emailAddress`, this test will instantly fail, outputting a precise `ajv` error telling you exactly which property is missing from the contract!

---

## The End of a Journey

Congratulations! Over the course of 114 tutorials, you have evolved from basic Selenium bindings into a Master Automation Architect. 

You have built:
* **UI Foundation**: ChromeDriver instantiation, explicit waits, locators.
* **Architecture**: Page Object Models, Singleton Factories, Screenplay patterns.
* **Utilities**: ConfigManagers, Winston Logging, Custom Jest Reporters.
* **Backend Hybrid**: Axios API Wrappers, JSON Schema Validations.

You now possess the skills to walk into any Enterprise organization and architect a scalable, robust, and state-of-the-art Test Automation Framework from scratch.

Thank you for following along on this incredible journey at MyCodeYatra!
