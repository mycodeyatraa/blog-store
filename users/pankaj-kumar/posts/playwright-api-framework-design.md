---
title: "API Automation Framework Design"
date: "2025-03-25"
description: "Learn how to structure a clean, scaleable API testing framework in Playwright using the Service Object Model (SOM) to encapsulate request logic and endpoints."
tags: ["Playwright", "TypeScript", "API Testing", "Framework Design"]
---

Welcome to Blog 42 of the **Playwright TypeScript Mastery Series**!

When you start writing many API tests, putting raw URL strings and raw payload mappings inside your test files quickly leads to duplicated code and maintenance nightmares. Just like UI testing uses the Page Object Model (POM) to decouple UI structure from tests, API testing uses the **Service Object Model (SOM)**.

Today, we will learn how to design an enterprise-grade API testing framework in Playwright by building reusable service classes that encapsulate endpoint configurations and request logic.

---

### What is the Service Object Model (SOM)?

The Service Object Model abstracts HTTP operations (GET, POST, etc.) for a specific domain entity into a dedicated class.
- **Service Layer**: Houses the URL routes, headers, payload constructors, and API request calls.
- **Test Layer**: Focuses purely on setting test conditions and writing assertions.

If an endpoint path changes or a payload key is renamed, you only update it once in your Service class—your test specs remain untouched.

---

### Step 1: Create the API Service Layer

Create a directory called `api` and add a new file `api/post-service.ts`:

```typescript
import { APIRequestContext } from '@playwright/test';
 
export class PostService {
  private request: APIRequestContext;
 
  constructor(request: APIRequestContext) {
    this.request = request;
  }
 
  /**
   * Retrieves a post by its ID
   * @param id The post identifier
   */
  async getPost(id: number) {
    return await this.request.get(`https://jsonplaceholder.typicode.com/posts/${id}`);
  }
 
  /**
   * Creates a new post resource
   * @param payload The request body object containing post details
   */
  async createPost(payload: { title: string; body: string; userId: number }) {
    return await this.request.post('https://jsonplaceholder.typicode.com/posts', {
      data: payload,
      headers: {
        'Content-Type': 'application/json; charset=UTF-8'
      }
    });
  }
 
  /**
   * Deletes a post resource by its ID
   * @param id The post identifier to remove
   */
  async deletePost(id: number) {
    return await this.request.delete(`https://jsonplaceholder.typicode.com/posts/${id}`);
  }
}
```

---

### Step 2: Implement the Test Spec

Now, write your tests in `tests/blog42_api_framework.spec.ts` using our newly created service class:

```typescript
import { test, expect } from '@playwright/test';
import { PostService } from '../api/post-service';
 
test.describe('Blog 42: API Testing Framework Design', () => {
  let postService: PostService;
 
  // Initialize service objects before each test using the default request fixture
  test.beforeEach(({ request }) => {
    postService = new PostService(request);
  });
 
  test('Should fetch post using service object model', async () => {
    const response = await postService.getPost(1);
    expect(response.status()).toBe(200);
 
    const responseBody = await response.json();
    expect(responseBody.id).toBe(1);
    console.log('[API Framework] Retrieved post successfully using Service Object Model.');
  });
 
  test('Should create post using service object model', async () => {
    const payload = {
      title: 'API Framework Design',
      body: 'Decoupling endpoints from tests improves code reusability.',
      userId: 1
    };
 
    const response = await postService.createPost(payload);
    expect(response.status()).toBe(201);
 
    const responseBody = await response.json();
    expect(responseBody.title).toBe(payload.title);
    console.log('[API Framework] Created post successfully using Service Object Model.');
  });
 
});
```

---

### Step 3: Run the Test

Execute the test suite using the Playwright CLI:

```
npx playwright test tests/blog42_api_framework.spec.ts
```

**Output:**

```
Running 2 tests using 1 worker
 
[API Framework] Retrieved post successfully using Service Object Model.
  ✓  1 tests/blog42_api_framework.spec.ts:12:7 › Blog 42: API Testing Framework Design › Should fetch post using service object model (115ms)
[API Framework] Created post successfully using Service Object Model.
  ✓  2 tests/blog42_api_framework.spec.ts:21:7 › Blog 42: API Testing Framework Design › Should create post using service object model (630ms)
 
  2 passed (1.9s)
```

In the next blog, we will step into **Phase 6** of our series and learn how to handle **Database Testing and Integrations** in Playwright!
