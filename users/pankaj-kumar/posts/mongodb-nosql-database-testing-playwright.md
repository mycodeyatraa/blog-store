---
title: MongoDB NoSQL Validation in Playwright
date: 29-Apr-2025
lastUpdated: 29-Apr-2025
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: ["playwright", "typescript", "database", "mongodb", "nosql", "full-stack"]
category: Enterprise Validation
categories: ["Enterprise Validation", "UI Automation", "Playwright", "TypeScript", "Database"]
excerpt: >-
  Learn how to natively query MongoDB clusters, manipulate BSON documents, and seamlessly bypass UI setup steps using the official MongoDB Node.js driver.
readTime: 5 min read
---

While SQL databases are standard, modern Enterprise stacks increasingly rely on NoSQL document databases like **MongoDB**. 

Because NoSQL databases store data in flexible, JSON-like formats (BSON), they are incredibly natural to work with inside Playwright's TypeScript ecosystem. 

In this tutorial, we will look at how to connect to a MongoDB cluster, assert JSON document states natively, and handle asynchronous UI test seeding.

### 1. Installation

Install the official MongoDB Node.js driver. This driver is fully typed for TypeScript out of the box.

```bash
npm install mongodb
```

### 2. Managing the Client Connection

Just like SQL connection pools, you want to instantiate your `MongoClient` once inside the `test.beforeAll` hook to prevent performance bottlenecks.

**File:** `tests/mongodb.spec.ts`

```typescript
import { test, expect } from '@playwright/test';
import { MongoClient, Db, Collection } from 'mongodb';
 
const MONGO_URI = 'mongodb://localhost:27017';
const DB_NAME = 'testdb';
 
test.describe('MongoDB Validation', () => {
  let client: MongoClient;
  let db: Db;
  let usersCollection: Collection;
 
  test.beforeAll(async () => {
    // 1. Establish the MongoDB connection once for the entire suite
    client = new MongoClient(MONGO_URI);
    await client.connect();
    db = client.db(DB_NAME);
    usersCollection = db.collection('users');
  });
 
  test.afterAll(async () => {
    // 2. Safely close the connection pool when tests finish
    if (client) {
      await client.close();
    }
  });
  
  // ... tests ...
});
```

### 3. Asserting Nested Document State

When an API creates a MongoDB document, it usually nests data deeply. Querying this in Node.js is natively supported without needing to write complex JOIN statements.

```typescript
  test('Assert Backend State after API execution', async ({ request }) => {
    const orderId = 'ORD-MONGO-999';
 
    // 1. Simulate an API action that creates a document
    const response = await request.post('/api/orders', { data: { id: orderId, amount: 500 } });
    expect(response.ok()).toBeTruthy();
 
    // 2. Query the MongoDB collection
    const order = await db.collection('orders').findOne({ id: orderId });
    
    // 3. Assert against the NoSQL Document structure seamlessly
    expect(order).not.toBeNull();
    expect(order?.status).toBe('PROCESSED');
    expect(order?.metadata?.source).toBe('API');
    
    // Teardown natively
    await db.collection('orders').deleteOne({ id: orderId });
  });
```

### 4. Injecting Complex Test State

Injecting UI test setups into a relational SQL database usually requires writing queries across 5 different tables. With MongoDB, you inject the entire application state in one single line.

```typescript
  test('Seed NoSQL Document and Validate via UI', async ({ page }) => {
    const testUser = {
      _id: 'user_nosql_123',
      username: 'mongo_tester',
      roles: ['admin', 'editor'],
      preferences: { theme: 'dark', notifications: true }
    };
 
    try {
      // 1. Seed a complex nested JSON document natively
      await usersCollection.insertOne(testUser);
 
      // 2. Perform UI validation assuming the data exists
      await page.goto('/login');
      await page.fill('#username', testUser.username);
      await page.click('#submit');
      
      await expect(page.locator('.welcome')).toHaveText('Welcome, mongo_tester');
 
    } finally {
      // 3. Clean up the document natively using a `try/finally` block
      // to ensure cleanup happens even if the UI assertion fails!
      await usersCollection.deleteOne({ _id: testUser._id });
    }
  });
```

### Summary

Integrating MongoDB directly into your Playwright framework is a superpower. It allows you to construct complex, nested test scenarios in a single `.insertOne()` command, entirely bypassing brittle UI setup steps!
