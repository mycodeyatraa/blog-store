---
title: Mastering GraphQL API Testing in Playwright
date: 30-Apr-2025
lastUpdated: 30-Apr-2025
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: ["playwright", "typescript", "graphql", "api", "testing", "backend"]
category: Enterprise Validation
categories: ["Enterprise Validation", "UI Automation", "Playwright", "TypeScript", "API Testing"]
excerpt: >-
  Learn how to natively validate enterprise GraphQL architectures, execute dynamic queries, and trigger data mutations using Playwright's built-in API context.
readTime: 4 min read
---

As modern Enterprise architectures shift from traditional REST APIs to **GraphQL**, your automation framework needs to adapt.

Testing GraphQL APIs might seem intimidating at first, but under the hood, a GraphQL request is simply a standard HTTP `POST` request sent to a single endpoint (usually `/graphql`). The payload is just a JSON object containing a `query` string and an optional `variables` object.

Because Playwright has a native `APIRequestContext` built-in, testing GraphQL is incredibly seamless!

### 1. Basic GraphQL Queries

Let's write a test that hits a public GraphQL API to fetch some data.

**File:** `tests/graphql.spec.ts`

```typescript
import { test, expect } from '@playwright/test';
 
const GRAPHQL_ENDPOINT = 'https://api.spacex.land/graphql/';
 
test('Execute a simple GraphQL Query', async ({ request }) => {
  // 1. Define the GraphQL Query as a string
  const query = `
    query GetCompanyInfo {
      company {
        name
        founder
        founded
      }
    }
  `;
 
  // 2. Send the POST request to the GraphQL endpoint
  const response = await request.post(GRAPHQL_ENDPOINT, {
    data: {
      query: query
    }
  });
 
  expect(response.ok()).toBeTruthy();
 
  // 3. Parse the JSON response
  const body = await response.json();
 
  // 4. Assert the nested GraphQL data payload
  expect(body.data.company.name).toBe('SpaceX');
  expect(body.data.company.founder).toBe('Elon Musk');
});
```

### 2. Parameterized Queries using Variables

Hardcoding values inside GraphQL query strings is a bad practice. Instead, you should always use the `variables` object to pass dynamic test data into your queries.

```typescript
test('Execute a parameterized GraphQL Query with Variables', async ({ request }) => {
  // 1. Define the parameterized Query using $ syntax
  const query = `
    query GetMission($id: ID!) {
      mission(id: $id) {
        name
        manufacturers
      }
    }
  `;
 
  // 2. Define the Query Variables dynamically
  const variables = {
    id: '9D1B7E0' // e.g., injected from a previous test step
  };
 
  // 3. Send both Query and Variables in the JSON payload
  const response = await request.post(GRAPHQL_ENDPOINT, {
    data: {
      query: query,
      variables: variables
    }
  });
 
  const body = await response.json();
  
  // 4. Assert the dynamic response
  expect(body.data.mission.name).toBe('Thaicom');
});
```

### 3. GraphQL Mutations (Data Insertion)

In GraphQL, retrieving data is called a `Query`, and modifying data (POST/PUT/DELETE) is called a `Mutation`. From Playwright's perspective, they are executed exactly the same way!

```typescript
test('Execute a GraphQL Mutation', async ({ request }) => {
  // 1. Define the Mutation
  const mutation = `
    mutation InsertUser($name: String!) {
      insert_users(objects: {name: $name}) {
        returning {
          id
          name
        }
      }
    }
  `;
 
  // 2. Execute the Mutation
  const response = await request.post('https://your-graphql-api.com/v1/graphql', {
    headers: { 'Authorization': `Bearer ${process.env.API_TOKEN}` },
    data: { 
      query: mutation, 
      variables: { name: 'Automation User' } 
    }
  });
  
  const body = await response.json();
  
  // Assert the newly created record ID exists
  expect(body.data.insert_users.returning[0].id).toBeDefined();
});
```

### Summary

Testing GraphQL requires no extra libraries or complex plugins in Playwright. By leveraging the native `request.post()` method and structuring your payload with `query` and `variables`, you can execute robust end-to-end integration tests against any GraphQL backend!
