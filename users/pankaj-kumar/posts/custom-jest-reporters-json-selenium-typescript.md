---
title: Extracting Test Metrics: Custom Jest Reporters
date: 30-Nov-2024
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [typescript, selenium, jest, reporters, metrics]
category: Selenium TypeScript
categories: [Selenium TypeScript, Framework Utilities & Configuration]
excerpt: >-
  Management wants dashboards! Learn how to intercept the Jest execution lifecycle and build a Custom Reporter to export JSON metrics.
readTime: 5 min read
---

# Extracting Test Metrics: Custom Jest Reporters

As your automation framework grows into an Enterprise solution, management will inevitably ask for dashboards. They want to know:
* "How many tests passed last night?"
* "Which tests took the longest to run?"
* "What is our failure rate?"

By default, Jest only outputs these metrics to the terminal. If you want to export this data to Elasticsearch, Datadog, or an HTML Dashboard, you need to intercept Jest's execution lifecycle.

We do this by building a **Custom Jest Reporter**.

---

## 1. Creating the Custom Reporter Class

A Custom Reporter in Jest is simply a JavaScript class with specific lifecycle methods like `onRunComplete` or `onTestResult`.

Let's build a Reporter that extracts the Start Time, Pass/Fail counts, and the execution duration for every test suite, saving it all into a clean `custom-report.json` file.

Create `tests/utils/CustomReporter.js`:
*(Note: Reporters run in the Node context before TypeScript compilation, so pure `.js` is often easiest!)*

```javascript
const fs = require('fs');
const path = require('path');
class CustomReporter {
  constructor(globalConfig, options) {
    this._globalConfig = globalConfig;
    this._options = options;
  }
  // Intercept the final results when the entire suite finishes!
  onRunComplete(contexts, results) {
    console.log('Custom Reporter: Run complete. Generating JSON report...');
    const reportPath = path.resolve(__dirname, '../../logs/custom-report.json');
    // Create a simplified, readable payload
    const simplifiedResults = {
      startTime: new Date(results.startTime).toISOString(),
      numTotalTests: results.numTotalTests,
      numPassedTests: results.numPassedTests,
      numFailedTests: results.numFailedTests,
      testSuites: results.testResults.map(suite => ({
        filePath: suite.testFilePath,
        status: suite.numFailingTests > 0 ? 'FAILED' : 'PASSED',
        durationMs: suite.perfStats.end - suite.perfStats.start
      }))
    };
    // Save it to our logs directory
    fs.writeFileSync(reportPath, JSON.stringify(simplifiedResults, null, 2));
    console.log(`Report successfully written to ${reportPath}`);
  }
}
module.exports = CustomReporter;
```

---

## 2. Registering the Reporter in Jest Config

Now, we must tell Jest to use our new Reporter alongside the default terminal output.

Open `jest.config.js` and add the `reporters` array:

```javascript
/** @type {import('ts-jest').JestConfigWithTsJest} */
module.exports = {
  preset: 'ts-jest',
  testEnvironment: 'node',
  testMatch: ['**/tests/**/*.test.ts'],
  testTimeout: 60000,
  verbose: true,
  maxWorkers: 3,
  // Register our Custom Reporter!
  reporters: [
    "default",
    "<rootDir>/tests/utils/CustomReporter.js"
  ],
};
```

---

## 3. Test Execution & Output

Run your framework as usual:

```bash
> jest tests/framework_architecture.test.ts
```

In the terminal, you will see Jest's default output, followed by our Custom Reporter:

```text
Custom Reporter: Run complete. Generating JSON report...
Report successfully written to /logs/custom-report.json
```

If you open `logs/custom-report.json`, you will find your beautiful, parsed metrics ready to be consumed by any dashboarding tool:

```json
{
  "startTime": "2026-11-30T10:00:00.000Z",
  "numTotalTests": 1,
  "numPassedTests": 1,
  "numFailedTests": 0,
  "testSuites": [
    {
      "filePath": "C:\\mcyt-sel-typescript\\tests\\framework_architecture.test.ts",
      "status": "PASSED",
      "durationMs": 4532
    }
  ]
}
```

## Conclusion

Custom Reporters allow you to bridge the gap between Automation Execution and Enterprise Monitoring. You could easily modify the `onRunComplete` method to HTTP POST this JSON payload directly into a database or Slack channel!

This absolutely concludes **Phase 4**! 
Next, we will begin **Phase 5**, where we introduce **API Testing Integration** using Axios and SuperTest!
