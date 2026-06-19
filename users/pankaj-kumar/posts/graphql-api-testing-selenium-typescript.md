---
title: Bypassing the REST: GraphQL Testing
date: 21-Jan-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [typescript, selenium, graphql, api-testing, enterprise-validation, axios, backend]
category: Selenium TypeScript
categories: [Selenium TypeScript, Enterprise Validation]
excerpt: >-
  Overcome the limitations of REST. Learn how to write GraphQL Queries and Mutations using Axios, instantly setup test data, and assert backend logic directly in your Selenium TypeScript framework.
readTime: 4 min read
---

# Bypassing the REST: GraphQL Testing

For the last decade, REST has been the dominant architecture for APIs. When a Selenium test needed to create test data, you would simply send a `POST` request to `/api/users`.

But REST has limitations. It often suffers from **Over-fetching** (returning too much data) and **Under-fetching** (requiring multiple API calls to get nested relationships).

To solve this, Facebook created **GraphQL**. 

GraphQL exposes a single endpoint (usually `/graphql`). Instead of hitting different URLs, you send a highly specific "Query" (to read data) or "Mutation" (to write data). Today, we will integrate GraphQL assertions into our Selenium TypeScript framework.

---

## 1. Setting Up Axios for GraphQL

To interact with a GraphQL API in Node.js, we don't need a massive, heavy GraphQL client like Apollo. We can use our old friend, `axios`.

Install axios:

```bash
npm install axios
```

---

## 2. Creating the GraphQL Manager

In GraphQL, every request is a `POST` request to the `/graphql` endpoint. The body of the request contains the `query` and optionally some `variables`.

Let's create a generic `GraphQLClient.ts` utility:

```typescript
import axios from 'axios';
import * as dotenv from 'dotenv';
dotenv.config();
export class GraphQLClient {
  private static readonly endpoint = process.env.GRAPHQL_URL || 'http://localhost:4000/graphql';
  static async send(query: string, variables?: Record<string, any>) {
    const response = await axios.post(
      this.endpoint,
      {
        query: query,
        variables: variables,
      },
      {
        headers: {
          'Content-Type': 'application/json',
          'Authorization': `Bearer ${process.env.TEST_API_TOKEN}`,
        },
      }
    );
    // GraphQL returns a 200 OK even if there are errors!
    // We must check the 'errors' array in the response body.
    if (response.data.errors) {
      throw new Error(`GraphQL Error: ${JSON.stringify(response.data.errors)}`);
    }
    return response.data.data;
  }
}
```

---

## 3. Writing Queries and Mutations

We will abstract our GraphQL operations into a DAO, exactly like we did for our databases. 

Let's say we are testing a Book Store application. We want to test that a user can add a book to their cart. We need a `Mutation` to create a book for the test, and a `Query` to verify it.

Create `BookGraphQLDAO.ts`:

```typescript
import { GraphQLClient } from '../utils/GraphQLClient';
export class BookGraphQLDAO {
  // A Mutation modifies data
  static async createTestBook(title: string, authorId: string): Promise<string> {
    const mutation = `
      mutation CreateBook($title: String!, $authorId: ID!) {
        addBook(title: $title, authorId: $authorId) {
          id
          title
        }
      }
    `;
    const variables = { title, authorId };
    const response = await GraphQLClient.send(mutation, variables);
    return response.addBook.id; // Return the newly created ID
  }
  // A Query reads data
  static async getBookDetails(bookId: string) {
    const query = `
      query GetBook($id: ID!) {
        book(id: $id) {
          title
          price
          stockCount
        }
      }
    `;
    const response = await GraphQLClient.send(query, { id: bookId });
    return response.book;
  }
}
```

---

## 4. The E2E GraphQL Test

Now, we write an Enterprise Validation test. 

We will use a GraphQL Mutation to generate our test data instantly (much faster than clicking through the UI to create a book). Then, we will use Selenium to buy the book. Finally, we will use a GraphQL Query to verify the stock count decreased.

```typescript
import { Builder, WebDriver } from 'selenium-webdriver';
import { expect } from 'chai';
import { StorePage } from '../pages/StorePage';
import { BookGraphQLDAO } from '../api/BookGraphQLDAO';
describe('GraphQL Inventory Validation', function () {
  let driver: WebDriver;
  let testBookId: string;
  before(async function () {
    // 1. Setup Data via GraphQL Mutation!
    testBookId = await BookGraphQLDAO.createTestBook('Automating GraphQL', 'author_123');
    driver = await new Builder().forBrowser('chrome').build();
  });
  after(async function () {
    await driver.quit();
    // Teardown: You would add a deleteBook mutation here
  });
  it('should decrease stock count when purchased via UI', async function () {
    const storePage = new StorePage(driver);
    await storePage.navigate();
    // 2. UI Layer Action
    await storePage.searchBook('Automating GraphQL');
    await storePage.clickBuyNow();
    await storePage.completeCheckout();
    // 3. UI Layer Assertion
    const receipt = await storePage.getReceiptMessage();
    expect(receipt).to.include('Purchase Successful');
    // 4. API Layer Assertion (GraphQL Query)
    const bookData = await BookGraphQLDAO.getBookDetails(testBookId);
    // Verify the backend correctly updated the inventory
    expect(bookData).to.not.be.null;
    expect(bookData.stockCount).to.equal(0);
  });
});
```

## Conclusion

By treating GraphQL as an extension of our Data Access Layer, we gain immense power. We can use Mutations to setup/teardown test state instantly, bypassing brittle UI flows. And we can use Queries to validate complex backend relationships without touching a database driver.

But databases and APIs are synchronous. You request data, and you get an immediate response. 

What happens when your application architecture is entirely asynchronous? What happens when you click a button and an event is fired into the void?

In the next tutorial, we tackle the hardest Enterprise topic of all: **Kafka Validation**.
