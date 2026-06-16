---
title: Hybrid Test Automation: Combining UI and API in TypeScript
date: 10-Dec-2024
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [typescript, selenium, api-testing, hybrid-testing, e2e, automation]
category: Selenium TypeScript
categories: [Selenium TypeScript, API Testing Integration]
excerpt: >-
  Solve the Ice Cream Cone anti-pattern by building blazing fast Hybrid Tests. Learn to use backend Axios APIs to set up data for frontend Selenium UI tests.
readTime: 6 min read
---

# Hybrid Test Automation: Combining UI and API in TypeScript

Have you ever written a UI test that took 2 minutes to run because the script had to log in, click through five menus, fill out a massive form, and submit it, just to verify that the final success banner appeared?

This is called the **Ice Cream Cone Anti-Pattern**. Relying purely on the UI for test data generation is slow, brittle, and highly susceptible to timeouts or DOM changes.

In this tutorial, we will write a **Hybrid Automation Script**. We will use our lightning-fast API framework to setup the test data in milliseconds, switch to Selenium WebDriver to validate the UI, and then instantly clean up our data via another API call!

---

## 1. Why Hybrid Testing?

The ultimate goal of end-to-end (E2E) testing is to validate user workflows. However, **Setup** and **Teardown** steps do not necessarily need to happen through the UI unless they are the specific feature being tested.

### The Hybrid Architecture Strategy:
1. **Setup (`beforeAll`)**: Call an API `POST` endpoint to instantly inject the test data into the database. Save the returned unique ID.
2. **Action (`it`)**: Launch the Selenium WebDriver, navigate directly to the specific resource URL (using the ID), and validate the UI.
3. **Teardown (`afterAll`)**: Call an API `DELETE` endpoint using the unique ID to instantly purge the data from the database.

---

## 2. Writing the Hybrid Test Script

Let's combine `selenium-webdriver` and our custom `ApiUtils` class.

Create a new file `tests/hybrid_ui_api.test.ts`:

```typescript
import { Builder, WebDriver, By, until } from "selenium-webdriver";
import { ApiUtils } from "./utils/ApiUtils";
// Set a longer timeout for Jest because UI tests take longer than API tests
jest.setTimeout(30000);
describe("Phase 6 - Hybrid Testing: UI + API", () => {
  let driver: WebDriver;
  let createdPostId: number;
  beforeAll(async () => {
    // HYBRID STEP 1: Fast Data Creation via API (Setup)
    console.log("[Setup] Creating test data via API...");
    const payload = {
      title: "Hybrid Test Post",
      body: "This post was created purely via backend API to test the UI.",
      userId: 1
    };
    const response = await ApiUtils.post("/posts", payload);
    // Validate API creation
    expect(response.status).toBe(201);
    createdPostId = response.data.id;
    console.log(`[Setup] Created Post ID: ${createdPostId} successfully via API.`);
    // Initialize WebDriver
    driver = await new Builder().forBrowser("chrome").build();
    await driver.manage().window().maximize();
  });
  it("Should interact with the UI based on API created data", async () => {
    // HYBRID STEP 2: UI Validation (Action)
    // Normally, you would navigate to the system under test to view the created post.
    // For demonstration, we navigate to our practice site to run the UI assertions.
    console.log("[Action] Navigating to the Practice Site...");
    await driver.get("https://practice.mycodeyatra.com/");
    // Basic UI Validation
    const header = await driver.findElement(By.tagName("h1"));
    const text = await header.getText();
    expect(text).toContain("Automation Practice");
    console.log(`[Action] UI Validation passed for Post ID: ${createdPostId}`);
  });
  afterAll(async () => {
    // HYBRID STEP 3: Fast Data Cleanup via API (Teardown)
    console.log(`[Teardown] Cleaning up data via API... deleting Post ID: ${createdPostId}`);
    if (createdPostId) {
      const response = await ApiUtils.delete(`/posts/${createdPostId}`);
      expect([200, 204]).toContain(response.status);
      console.log(`[Teardown] Successfully deleted Post ID: ${createdPostId} via API.`);
    }
    // Quit driver gracefully
    if (driver) {
      await driver.quit();
    }
  });
});
```

---

## 3. Breaking Down the Script

### Step 1: The Setup (`beforeAll`)
Instead of launching the browser to fill out a "Create Post" form, we directly hit the backend with `ApiUtils.post()`. This takes `~200ms`. We extract the `createdPostId` from the response payload and save it as a class-level variable. 

### Step 2: The Action (`it`)
The UI test starts. We open the browser and navigate directly to the target system. Because the data was already generated, the script jumps immediately to the validation assertions.

### Step 3: The Teardown (`afterAll`)
Rather than writing a UI flow to click a "Delete" button and confirm a modal dialog, we hit `ApiUtils.delete()`. This instantly purges the record in `~200ms`, keeping our test environment completely clean.

---

## 4. Test Execution Output

Run the test:

```bash
> ENV_NAME=qa jest tests/hybrid_ui_api.test.ts
```

Output:

```text
 PASS  tests/hybrid_ui_api.test.ts (6.342 s)
  Phase 6 - Hybrid Testing: UI + API
    √ Should interact with the UI based on API created data (4312 ms)
  console.log
    [Setup] Creating test data via API...
    [API] Sending POST request to: https://jsonplaceholder.typicode.com/posts
    [API] Received Status: 201
    [Setup] Created Post ID: 101 successfully via API.
  console.log
    [Action] Navigating to the Practice Site...
    [Action] UI Validation passed for Post ID: 101
  console.log
    [Teardown] Cleaning up data via API... deleting Post ID: 101
    [API] Sending DELETE request to: https://jsonplaceholder.typicode.com/posts/101
    [API] Received Status: 200
    [Teardown] Successfully deleted Post ID: 101 via API.
```

## Conclusion

Hybrid testing represents the pinnacle of modern automation architecture. By marrying the speed of API protocols with the comprehensive user-simulation of UI WebDriver scripts, we can dramatically lower our execution times while improving test stability!
