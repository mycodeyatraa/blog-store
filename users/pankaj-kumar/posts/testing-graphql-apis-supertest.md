---
title: Testing GraphQL APIs
date: 25-Feb-2025
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [supertest, graphql, query, mutation, apollo, jest]
category: API Testing
categories: [API Testing, NodeJS, GraphQL, Automation]
excerpt: >-
  Demystify GraphQL test automation. Learn how to execute Queries and Mutations by passing schema strings into SuperTest HTTP POST payloads.
readTime: 4 min read
---

While traditional REST APIs have strict routing (like `/api/users`), the modern web is rapidly adopting **GraphQL**. 

In GraphQL, all requests are sent via a `POST` method to a single unified endpoint (typically `/graphql`). Instead of hitting different URL paths, the client sends a highly specific string detailing exactly which fields it wants to query or mutate.

Because SuperTest is fundamentally just an HTTP client, testing GraphQL APIs is completely seamless!

---

## 1. Testing a GraphQL Query (Fetching Data)

To fetch data in GraphQL, we send a `POST` request to the `/graphql` endpoint and pass a `query` string inside the payload. 

```typescript
import request from 'supertest';
describe('Testing GraphQL APIs with SuperTest', () => {
    const API_URL = 'http://localhost:8080';
    it('1. Execute a GraphQL Query', async () => {
        // A GraphQL Query is just a POST request where the payload contains a "query" string
        const graphqlPayload = {
            query: `
                query {
                    users {
                        id
                        name
                        role
                    }
                }
            `
        };
        const response = await request(API_URL)
            .post('/graphql')
            .send(graphqlPayload);
        expect(response.status).toBe(200);
        // GraphQL always encapsulates the response inside a 'data' object
        expect(response.body).toHaveProperty('data');
        expect(response.body.data.users).toBeInstanceOf(Array);
        expect(response.body.data.users[0]).toHaveProperty('name');
    });
```

Notice that we assert against `response.body.data`. According to the official GraphQL specification, all successful executions must return their JSON payload encapsulated inside a root `data` object.

## 2. Testing a GraphQL Mutation (Modifying Data)

When you want to create, update, or delete data in GraphQL, you execute a **Mutation**. The methodology in SuperTest is exactly the same as querying!

```typescript
    it('2. Execute a GraphQL Mutation', async () => {
        // Mutations are used to modify data
        const mutationPayload = {
            query: `
                mutation {
                    createUser(name: "GraphQL User", email: "gql@mycodeyatra.com", role: "admin") {
                        id
                        name
                        email
                    }
                }
            `
        };
        const response = await request(API_URL)
            .post('/graphql')
            .send(mutationPayload);
        expect(response.status).toBe(200);
        expect(response.body.data.createUser.name).toBe('GraphQL User');
        expect(response.body.data.createUser.email).toBe('gql@mycodeyatra.com');
    });
});
```

## 3. Execution Results

Let's execute the suite via `npx jest tests/graphql.test.ts` to see our tests hit the Apollo GraphQL server.

```bash
PASS tests/graphql.test.ts
  Testing GraphQL APIs with SuperTest
    [PASS] 1. Execute a GraphQL Query (140 ms)
    [PASS] 2. Execute a GraphQL Mutation (14 ms)
Test Suites: 1 passed, 1 total
Tests:       2 passed, 2 total
Snapshots:   0 total
Time:        3.739 s
```

## 4. Conclusion

Because GraphQL operates entirely over standard HTTP POST requests, you don't need any specialized tools to test it. SuperTest's fluent `.send()` architecture handles the payload translation beautifully. 

In the next article, we will pivot to exploring WebSockets and how to test bidirectional persistent connections!
