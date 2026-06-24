---
title: Advanced Step Definitions: DataTables and Outlines
date: 20-Apr-2025
lastUpdated: 20-Apr-2025
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: ["playwright", "typescript", "bdd", "cucumber", "step-definitions", "datatables"]
category: UI Automation
categories: ["UI Automation", "Playwright", "TypeScript", "BDD", "Cucumber"]
excerpt: >-
  Supercharge your BDD automation by learning how to parse Gherkin Data Tables and dynamically extract parameter variables into Playwright TypeScript.
readTime: 5 min read
---

Writing basic step definitions for a simple `Given/When/Then` scenario is straightforward. But enterprise applications require testing with complex data structures, dynamic inputs, and repetitive hooks. 

In this tutorial, we will write advanced Cucumber Step Definitions to handle `Background` hooks, parameter extraction, and `DataTable` parsing in TypeScript.

### 1. Handling Dynamic Parameters

In our previous Gherkin syntax tutorial, we utilized a `Scenario Outline` with an `Examples` table. Cucumber passes these values down into your TypeScript functions dynamically. 

By using the `{string}` expression, we instruct Cucumber to capture any text wrapped in quotes and pass it as an argument:

```typescript
// features/advanced-gherkin.feature:
// Given the user has added a "Laptop" priced at "$1000" to the cart
 
import { Given } from '@cucumber/cucumber';
 
Given('the user has added a {string} priced at {string} to the cart', async (product: string, price: string) => {
    console.log(`Added ${product} at ${price}`);
    // Await Playwright UI Interactions here...
});
```

Because this step is part of a `Scenario Outline`, this identical TypeScript function will execute three times automatically, injecting the values from the `Examples` table on each iteration!

### 2. Parsing Data Tables

When you need to pass a list of complex objects (like multiple items in a shopping cart), Gherkin `Data Tables` are incredibly useful. Cucumber injects these tables into your TypeScript code as a special `DataTable` object.

You can easily convert this object into an array of JSON objects using `dataTable.hashes()`:

```typescript
// features/advanced-gherkin.feature:
// Given the user adds the following items to the cart:
//   | Item Name | Quantity | Price |
//   | Mouse     | 2        | $25   |
 
import { Given, DataTable } from '@cucumber/cucumber';
 
Given('the user adds the following items to the cart:', async (dataTable: DataTable) => {
    // Convert the data table to an array of objects mapping to the headers
    const items = dataTable.hashes();
 
    for (const item of items) {
        console.log(`Processed Item: ${item['Item Name']} | Qty: ${item['Quantity']} | Price: ${item['Price']}`);
 
        // Example Playwright usage:
        // await page.fill('#search', item['Item Name']);
        // await page.click('#add-to-cart');
    }
});
```

### 3. Execution Results

Let's execute our `advanced-gherkin.feature` file using the `npx cucumber-js` CLI. Notice how the single Data Table definition successfully processes all three rows in a tight loop!

```text
Processed Item: Mouse | Qty: 2 | Price: $25
Processed Item: Keyboard | Qty: 1 | Price: $50
Processed Item: USB-C Cable | Qty: 3 | Price: $10
Proceeding to checkout page
Asserting total checkout amount is: $130
 
...
9 scenarios (9 passed)
50 steps (50 passed)
0m 12.524s (0m 13.743s executing your code)
```

### Summary

Advanced step definitions are the powerhouse of BDD automation. By leveraging Cucumber's native `{string}` expressions and `DataTable.hashes()` methods, your Playwright TypeScript framework can process massive amounts of complex data without writing brittle, hardcoded loops.
