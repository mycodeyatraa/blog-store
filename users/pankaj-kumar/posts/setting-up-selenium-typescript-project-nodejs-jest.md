---
title: Setting Up Your Selenium TypeScript Project (Node.js, Jest, and WebDriver)
date: 03-Nov-2024
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [typescript, selenium, jest, nodejs, configuration]
category: Selenium TypeScript
categories: [Selenium TypeScript, UI Automation, Architecture]
excerpt: >-
  Ready to code? Learn how to initialize a Node.js project, configure the TypeScript compiler, and write your very first Selenium automated browser test using the Jest testing framework.
readTime: 6 min read
---

# Setting Up Your Selenium TypeScript Project (Node.js, Jest, and WebDriver)

In our previous article, we discussed why TypeScript is rapidly becoming the standard language for Enterprise UI Automation. Now, it's time to build our environment from scratch!

By the end of this tutorial, you will have installed Node.js, initialized a TypeScript project, configured the Jest test runner, and written your very first Selenium automated browser test!

---

## 1. System Requirements

Before we write code, you must install the runtime environment:
1. **Install Node.js:** Go to [nodejs.org](https://nodejs.org) and download the LTS (Long Term Support) version. This will also install `npm` (Node Package Manager).
2. **Install VS Code:** Download [Visual Studio Code](https://code.visualstudio.com/), the undisputed best IDE for TypeScript development.

---

## 2. Initializing the Project

Open your terminal, create a new directory for your automation framework, and initialize a new Node project:

```bash
mkdir mcyt-sel-typescript
cd mcyt-sel-typescript
npm init -y
```
This generates a `package.json` file, which will keep track of all our framework's dependencies.

---

## 3. Installing Dependencies

We need three types of dependencies:
- **Selenium WebDriver:** The core browser automation library.
- **TypeScript:** The compiler and type definitions.
- **Jest:** The testing framework (like Pytest or TestNG) that will execute our test suites.

Run the following command to install everything:

```bash
npm install selenium-webdriver
npm install --save-dev typescript jest ts-jest @types/jest @types/selenium-webdriver
```

Notice how we installed `@types/selenium-webdriver` and `@types/jest`? These are the "Type Definition" files that give VS Code the incredible autocomplete and error-checking capabilities we love!

---

## 4. Configuring TypeScript and Jest

To tell the TypeScript compiler how to translate our `.ts` files into `.js` files, we need a `tsconfig.json` file. 

Create `tsconfig.json` in the root of your project:

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "CommonJS",
    "moduleResolution": "node",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "outDir": "./dist",
    "types": ["jest", "node"]
  },
  "include": ["tests/**/*", "src/**/*"],
  "exclude": ["node_modules"]
}
```

Next, configure Jest by creating `jest.config.js`:

```javascript
/** @type {import('ts-jest').JestConfigWithTsJest} */
module.exports = {
  preset: 'ts-jest',
  testEnvironment: 'node',
  testMatch: ['**/tests/**/*.test.ts'],
  testTimeout: 30000,
  verbose: true,
};
```
This configuration tells Jest to look inside the `tests` folder for any file ending in `.test.ts`.

---

## 5. Writing Your First TypeScript Automated Test

Let's write a simple script that opens Google Chrome, navigates to MyCodeYatra, and verifies the page title.

Create a folder named `tests`, and inside it, create `setup.test.ts`:

```typescript
import { Builder, WebDriver } from "selenium-webdriver";
describe("Selenium TypeScript Foundations", () => {
  let driver: WebDriver;
  // Runs ONCE before all tests in this block
  beforeAll(async () => {
    driver = await new Builder().forBrowser("chrome").build();
    await driver.manage().window().maximize();
  });
  // Runs ONCE after all tests are finished
  afterAll(async () => {
    if (driver) {
      await driver.quit();
    }
  });
  it("Should navigate to MyCodeYatra and verify title", async () => {
    // 1. Navigate to URL
    await driver.get("https://mycodeyatra.com");
    // 2. Fetch the Title
    const title = await driver.getTitle();
    console.log("Page Title:", title);
    // 3. Assert the Title is correct using Jest's 'expect'
    expect(title).toContain("MyCodeYatra");
  });
});
```

### The Power of `async / await`
Notice how almost every line of code inside the `it` block starts with `await`? 
JavaScript is fundamentally **asynchronous**. If you don't use the `await` keyword, Node.js will try to fetch the title before the browser has even finished navigating! You must always `await` WebDriver promises in TypeScript.

---

## 6. Running the Test

To run the test, update your `package.json` scripts block:

```json
  "scripts": {
    "test": "jest"
  }
```

Now, just type `npm test` in your terminal. Jest will automatically compile your TypeScript, launch Google Chrome, verify the title, close the browser, and print a beautiful green **PASS** in your console!

## Conclusion

Congratulations! You have successfully configured a modern TypeScript testing environment from scratch. You now have Node.js running Jest, compiling TypeScript dynamically via `ts-jest`, and executing native Selenium commands!

In our next article, we will dive deep into **Locators**, exploring how to find web elements safely using CSS Selectors and XPath, and how to interact with them securely.
