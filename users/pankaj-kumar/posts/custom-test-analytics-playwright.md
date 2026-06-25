---
title: Building Custom Test Analytics in Playwright
date: 12-May-2025
lastUpdated: 12-May-2025
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: ["playwright", "typescript", "analytics", "reporting", "nodejs", "metrics"]
category: Reporting
categories: ["Reporting", "UI Automation", "Playwright", "TypeScript", "Performance"]
excerpt: >-
  Parse Playwright's native JSON reporter output to build custom analytics scripts that detect your slowest bottlenecks and generate failure heatmaps across your repository.
readTime: 4 min read
---

As your Playwright repository scales to thousands of tests, simply looking at a green or red pipeline isn't enough. You need **Test Analytics** to answer engineering questions like:

1. *Which 5 tests are the absolute slowest and bottlenecking our CI/CD?*
2. *Which spec files are the most brittle (highest failure rate over time)?*
3. *Is our overall execution duration creeping upward?*

While Allure provides some of this, writing a custom Node.js analytics script allows you to extract precise metrics to feed into external alerting tools or generate custom Slack reports!

### 1. Generating the Raw JSON Data

Playwright includes a native `json` reporter that dumps the entire execution hierarchy—suites, specs, steps, durations, and statuses—into a single massive file.

You can configure this in your config file, or just append it via the CLI:

```bash
npx playwright test --reporter=json > report.json
```

### 2. Building an Analytics Parser

Once `report.json` is generated, we can write a simple Node.js script that parses the JSON tree to calculate actionable metrics.

**File:** `scripts/analytics.js`

```javascript
const fs = require('fs');
 
// Read the raw JSON dump
const report = JSON.parse(fs.readFileSync('report.json'));
 
let totalDuration = 0;
const testDurations = [];
const failureCountByFile = {};
 
// Recursively traverse the Playwright JSON tree structure
function processSuite(suite, fileName) {
  const currentFile = suite.file || fileName;
  
  for (const spec of suite.specs || []) {
    for (const test of spec.tests || []) {
      for (const result of test.results || []) {
        totalDuration += result.duration;
        
        // Track test speed
        testDurations.push({ title: spec.title, duration: result.duration });
 
        // Track flakiness/brittleness by file
        if (result.status === 'failed' || result.status === 'timedOut') {
          failureCountByFile[currentFile] = (failureCountByFile[currentFile] || 0) + 1;
        }
      }
    }
  }
 
  // Recurse into nested test.describe blocks
  for (const sub of suite.suites || []) {
    processSuite(sub, currentFile);
  }
}
 
// Start processing from the root suites
for (const s of report.suites || []) {
  processSuite(s, s.file);
}
```

### 3. Extracting the Insights

Now that the data is flattened, we can sort and filter it to generate our report!

```javascript
// 1. Identify the Top 5 Slowest Tests for Refactoring
const slowest = testDurations.sort((a, b) => b.duration - a.duration).slice(0, 5);
console.log('[Top 5 Slowest Tests]:');
slowest.forEach(t => console.log(`- ${t.title} (${t.duration}ms)`));
 
// 2. Identify the most Brittle Files (Failure Heatmap)
const heatmap = Object.entries(failureCountByFile).sort((a, b) => b[1] - a[1]);
console.log('\n[Failure Heatmap]:');
heatmap.forEach(([file, count]) => console.log(`- ${file}: ${count} failures`));
 
// 3. Execution Efficiency
console.log(`\nTotal Computation Time: ${(totalDuration / 1000).toFixed(2)}s`);
```

### Summary

By parsing Playwright's native JSON output, you unlock limitless custom analytics. You can pipe the output of this script into a `Slack Webhook` at the end of your CI/CD pipeline, instantly alerting the team to tests that have become dangerously slow or files that are historically brittle!
