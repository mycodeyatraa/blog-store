---
title: Generating Code Coverage & Test Reports with Jest & Supertest
date: 03-Mar-2026
lastUpdated: 03-Mar-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [supertest, jest, coverage, istanbul, nyc, reporting]
category: API Supertest
categories: [API Supertest, Node.js, Express]
excerpt: >-
  Measure API test quality! Configure Istanbul/nyc coverage reports and HTML test dashboards in Jest and Supertest.
readTime: 8 min read
---

# Generating Code Coverage & Test Reports with Jest & Supertest

Tracking test pass rates is only half the battle; engineering teams must also measure **Code Coverage** to ensure that API routes, controllers, middleware, and database access objects are thoroughly tested.

**Jest** comes equipped with built-in **Istanbul** coverage collection, generating HTML dashboards that highlight untested branches, functions, and statements.

---

## 1. Configuring Coverage Thresholds in `jest.config.js`

Add strict coverage thresholds to enforce quality gates:

**jest.config.js**

```javascript
module.exports = {
  preset: 'ts-jest',
  testEnvironment: 'node',
  collectCoverage: true,
  coverageDirectory: 'coverage',
  coverageReporters: ['text', 'lcov', 'html'],
  collectCoverageFrom: [
    'src/**/*.ts',
    '!src/server.ts' // Exclude entry point
  ],
  coverageThreshold: {
    global: {
      branches: 80,
      functions: 85,
      lines: 85,
      statements: 85
    }
  }
};
```

---

## 2. Generating Coverage & HTML Reports

Run your Supertest suite with coverage:

```bash
npx jest --coverage
```

The command generates interactive HTML reports in the `./coverage/lcov-report/index.html` directory, allowing teams to inspect exact line coverage visually.
