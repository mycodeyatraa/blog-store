---
title: Dynamic Test Data: Fixtures & Data Factories with Faker.js in Supertest
date: 01-Mar-2026
lastUpdated: 01-Mar-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: ["supertest", "fakerjs", "test-data", "factories", "fixtures"]
category: API Supertest
categories: ["API Supertest", "Node.js", "Express"]
excerpt: >-
  Eliminate hardcoded test data! Build modular data factories with @faker-js/faker and Supertest for dynamic API testing.
readTime: 8 min read
---

# Dynamic Test Data: Fixtures & Data Factories with Faker.js in Supertest

Hardcoding static strings (e.g., `"testuser123@example.com"`) into test cases causes unique constraint database collisions when executing test suites in parallel.

**Data Factories** leveraging **`@faker-js/faker`** generate unique, realistic test data dynamically (emails, names, credit cards, addresses) for every test run.

---

## 1. Setting Up @faker-js/faker

Install `@faker-js/faker` in your Node.js project:

```bash
npm install --save-dev @faker-js/faker
```

---

## 2. Building a Modular User Factory Class

Create a dedicated factory utility to produce customized request payloads:

**factories/userFactory.ts**

```typescript
import { faker } from '@faker-js/faker';
 
export interface UserPayload {
  firstName: string;
  lastName: string;
  email: string;
  phone: string;
  role: string;
}
 
export class UserFactory {
  static create(overrides: Partial<UserPayload> = {}): UserPayload {
    return {
      firstName: faker.person.firstName(),
      lastName: faker.person.lastName(),
      email: faker.internet.email(),
      phone: faker.phone.number(),
      role: 'CUSTOMER',
      ...overrides // Allows overriding specific fields
    };
  }
 
  static createAdmin(): UserPayload {
    return this.create({ role: 'ADMIN' });
  }
}
```

---

## 3. Executing Supertest API Tests with Dynamic Factories

**tests/user-registration.spec.ts**

```typescript
import request from 'supertest';
import app from '../src/app';
import { UserFactory } from '../factories/userFactory';
 
describe('User Registration with Dynamic Data Factory', () => {
 
  test('should create user with unique dynamic details', async () => {
    const newUser = UserFactory.create();
 
    const response = await request(app)
      .post('/api/v1/users')
      .send(newUser);
 
    expect(response.status).toBe(201);
    expect(response.body.email).toBe(newUser.email.toLowerCase());
    expect(response.body.role).toBe('CUSTOMER');
  });
 
  test('should create admin user using factory overrides', async () => {
    const adminUser = UserFactory.createAdmin();
 
    const response = await request(app)
      .post('/api/v1/users')
      .send(adminUser);
 
    expect(response.status).toBe(201);
    expect(response.body.role).toBe('ADMIN');
  });
});
```
