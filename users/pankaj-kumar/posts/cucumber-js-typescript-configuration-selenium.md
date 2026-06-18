---
title: Bringing Gherkin to Life: Installing and Configuring Cucumber-TS
date: 11-Jan-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [typescript, selenium, bdd, cucumber, cucumber-js, configuration, ts-node]
category: Selenium TypeScript
categories: [Selenium TypeScript, BDD & Cucumber Framework]
excerpt: >-
  Install @cucumber/cucumber and configure cucumber.js to dynamically compile TypeScript on the fly using ts-node, seamlessly executing your BDD feature files.
readTime: 4 min read
---

# Bringing Gherkin to Life: Installing and Configuring Cucumber-TS

For the past two tutorials, we have been writing plain English sentences in `.feature` files. As beautiful as they are, English sentences cannot click buttons in a Chrome browser.

To execute our Gherkin files, we need a runner. Enter **Cucumber**. 

Cucumber is the execution engine that reads `.feature` files, parses the Given/When/Then steps, and maps them to actual TypeScript functions that drive Selenium WebDriver.

Let's install and configure Cucumber for a TypeScript environment!

---

## 1. Installation

First, we need to install the core Cucumber library and its types:

```bash
npm install @cucumber/cucumber @types/cucumber --save-dev
```

Because Cucumber is fundamentally a Node.js test runner, it naturally expects JavaScript. Since we are writing our automation in TypeScript, we also need to ensure `ts-node` is available so Cucumber can compile our `.ts` files on the fly during execution:

```bash
npm install ts-node --save-dev
```

---

## 2. The Configuration File (cucumber.js)

When you type `npx cucumber-js` in the terminal, the engine immediately looks for a `cucumber.js` configuration file in the root of your project.

This file tells Cucumber exactly where to find your English features and where to find your TypeScript code.

Create a `cucumber.js` file in the root of your repository:

```javascript
module.exports = {
  default: {
    // 1. Where are the English Feature files?
    paths: ["tests/bdd/features/**/*.feature"],
    // 2. Tell Cucumber to use ts-node to compile TypeScript on the fly
    requireModule: ["ts-node/register"],
    // 3. Where is the actual TypeScript automation code?
    require: ["tests/bdd/steps/**/*.ts", "tests/bdd/support/**/*.ts"],
    // 4. How should the results be formatted?
    format: [
      "summary",
      "progress-bar",
      "html:reports/cucumber-report.html" // Generate a beautiful HTML report!
    ],
    // 5. Tell Cucumber we are using modern Async/Await syntax
    formatOptions: {
      snippetInterface: "async-await"
    },
    publishQuiet: true // Disable the annoying prompt to publish to the cloud
  }
};
```

### Breaking down the Config:
- **`paths`**: Points to the domain-driven folders we created in the last tutorial. The `/**/*.feature` glob pattern ensures it recursively finds every file in every subfolder.
- **`requireModule: ["ts-node/register"]`**: This is the magic bullet. Without this, Cucumber will crash when it sees TypeScript syntax (like `import` or type declarations).
- **`require`**: This points to the TypeScript files that actually contain our Selenium WebDriver code.
- **`format`**: We configure Cucumber to spit out a summary to the terminal, show a progress bar while running, and automatically generate a standalone HTML report when finished!

---

## 3. Modifying package.json

Finally, let's make it easy for our developers to execute the suite by adding a script to `package.json`:

```json
{
  "scripts": {
    "test:bdd": "cucumber-js"
  }
}
```

Now, we can simply run `npm run test:bdd`!

If you run this command right now, Cucumber will find the `intro.feature` file we wrote earlier, parse it, and print out warnings saying: `"Undefined. Implement with the following snippet..."`

## Conclusion

We now have the Cucumber engine installed and perfectly configured to compile TypeScript and read our Gherkin features! 

However, right now, Cucumber doesn't know *what* to do when it reads "Given the user logs in". 

In our next tutorial, we will write **Step Definitions**—the actual TypeScript functions that map directly to our Gherkin sentences and trigger Selenium WebDriver commands!
