---
title: Scenario Context and Custom Worlds in Cucumber TypeScript
date: 22-Apr-2025
lastUpdated: 22-Apr-2025
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: ["playwright", "typescript", "bdd", "cucumber", "architecture"]
category: UI Automation
categories: ["UI Automation", "Playwright", "TypeScript", "BDD", "Cucumber"]
excerpt: >-
  Eliminate global variables and safely share dynamic test data between disparate BDD steps using Cucumber's isolated Custom World context.
readTime: 4 min read
---

One of the most common challenges when building a scalable BDD framework is sharing data between entirely separate steps. For example, if a `Given` step generates a random user ID, how does the `Then` step know what that ID is without using messy global variables?

In Cucumber, the solution is the **Custom World Context**.

### What is the World?

In Cucumber, every single scenario runs in an isolated, dedicated context object known as the `World`. By default, it contains a few basic methods, but we can extend it to hold our custom application state!

### 1. Extending the World Context

First, we define an interface that extends the base `World` class. This class will hold the dynamic data we want to persist throughout the scenario lifecycle.

**File:** `tests/steps/context.steps.ts`

```typescript
import { Given, When, Then, setWorldConstructor, World, IWorldOptions } from '@cucumber/cucumber';
import { expect } from '@playwright/test';
 
// 1. Define a Custom World Interface extending Cucumber's default World
export class CustomWorld extends World {
    // This variable will hold our shared state across steps
    public trackingId: string | null = null;
 
    constructor(options: IWorldOptions) {
        super(options);
    }
}
 
// 2. Tell Cucumber to use our CustomWorld instead of the default one
setWorldConstructor(CustomWorld);
```

### 2. Binding Context to Steps (`this`)

To access the `CustomWorld` inside your step definitions, you must bind it to the execution context using the `this` keyword.

> **CRITICAL RULE**: You **cannot** use TypeScript arrow functions (`() => {}`) for step definitions if you want to use the World context. Arrow functions lexically bind `this` to the enclosing scope, preventing Cucumber from injecting the World object. You must use standard anonymous functions (`function() {}`).

```typescript
// 3. Use 'this' to read and write to the shared context!
Given('I generate a random tracking ID', async function (this: CustomWorld) {
    // Generate a dynamic ID
    this.trackingId = `TRK-${Math.floor(Math.random() * 1000000)}`;
    console.log(`[Given] Generated Tracking ID: ${this.trackingId}`);
});
 
When('I submit the order using the tracking ID', async function (this: CustomWorld) {
    // We safely read this.trackingId generated in the completely separate step above
    console.log(`[When] Submitting order payload with ID: ${this.trackingId}`);
    // Example: await page.fill('#tracking-input', this.trackingId!);
});
 
Then('the confirmation page should display the same tracking ID', async function (this: CustomWorld) {
    console.log(`[Then] Verifying UI displays ID: ${this.trackingId}`);
    // Assert the value persists
    expect(this.trackingId).not.toBeNull();
});
```

### Execution Output

When we run this scenario (`npx cucumber-js`), notice how the randomly generated tracking ID flawlessly persists across the three disparate step definitions:

```text
[Hook: Before] Starting Scenario: Passing data between Given, When, and Then
[Given] Generated Tracking ID: TRK-647395
[When] Submitting order payload with ID: TRK-647395
[Then] Verifying UI displays ID: TRK-647395
[Hook: After] Scenario Passing data between Given, When, and Then executed successfully.
```

### Summary

By leveraging `setWorldConstructor`, you eradicate the need for dangerous global variables (`let tempId = '';`). Every scenario gets a pristine, isolated `CustomWorld` instance, guaranteeing that massive parallel test suites won't experience data pollution!
