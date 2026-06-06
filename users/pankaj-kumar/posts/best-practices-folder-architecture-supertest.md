---
title: Best Practices & Folder Architecture
date: 06-Mar-2025
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [supertest, jest, architecture, best-practices, clean-code]
category: API Testing
categories: [API Testing, NodeJS, Software Architecture, Enterprise]
excerpt: >-
  Scale your API testing frameworks to thousands of tests. Discover the ultimate enterprise folder architecture and coding standards for SuperTest suites.
readTime: 5 min read
---

Congratulations! If you've followed this 16-part series, you now know everything from basic HTTP assertions to testing Graphql, WebSockets, File Uploads, Chaos Engineering, and CI/CD Pipelines.

However, writing good tests is only half the battle. As your SuperTest repository grows to encompass thousands of tests, the architecture of your repository will determine whether your team moves quickly, or drowns in technical debt.

In this final article, we will explore the **Ultimate SuperTest Folder Architecture**.

---

## 1. The Ideal Repository Structure

Instead of dumping everything into a single `tests/` directory, an enterprise-grade API test suite should decouple its configuration, data generation, and assertions.

Here is the recommended folder architecture:

```text
mcyt-api-supertest/
├── config/                 # Global environment configurations
│   ├── default.json        # Base URLs and default timeouts
│   └── custom-environment-variables.json
├── data/                   # JSON schemas, static files, and constants
│   ├── mock-users.json
│   └── sample.pdf          # For file upload testing
├── helpers/                # Reusable abstract functions
│   ├── auth.helper.ts      # Functions that automatically fetch JWTs
│   └── database.helper.ts  # Functions to seed/teardown databases
├── tests/                  # The actual Jest Test Files!
│   ├── auth/               # Grouped by API Controller/Domain
│   │   ├── login.test.ts
│   │   └── register.test.ts
│   ├── users/
│   │   ├── get-users.test.ts
│   │   └── create-user.test.ts
│   └── files/
│       └── upload.test.ts
├── jest.config.js          # Jest compiler rules and timeout boundaries
├── package.json
└── tsconfig.json
```

## 2. Key Best Practices

### A. Never Hardcode Base URLs
Do not write `http://localhost:8080` directly in your test files. Instead, use an environment variable (e.g., `process.env.API_URL`). This allows your CI/CD pipeline to point the exact same test suite at `staging.myapi.com` simply by swapping the env variable!

### B. Abstract Complex Setup Logic
If every single test file needs a User JWT to execute, do not write the login code 50 times. Abstract it into a helper function!

```typescript
// helpers/auth.helper.ts
import request from 'supertest';
export async function getAdminToken(apiUrl: string) {
    const res = await request(apiUrl)
        .post('/api/auth/login')
        .send({ email: 'admin', password: 'password123' });
    return res.body.token;
}
```

Now, your test files look incredibly clean:
```typescript
// tests/users/get-users.test.ts
beforeAll(async () => {
    token = await getAdminToken(process.env.API_URL);
});
```

### C. Use Faker.js Liberally
As we learned in Blog 11, never hardcode user emails, names, or IDs. By injecting Faker.js into your payloads, your test suite will passively fuzz your API for edge cases every single time it runs.

### D. Enforce Strict Linting and Formatting
Adopt `eslint` and `prettier`. In a large QA team, keeping the test code beautifully formatted is just as important as the production code.

## 3. Final Conclusion

SuperTest remains the absolute gold standard for API testing in the Node.js ecosystem. It perfectly bridges the gap between raw HTTP requests and expressive Assertions. 

Thank you for joining me on this 16-part deep dive into API Automation. You are now equipped to build, scale, and maintain enterprise-grade testing frameworks! Happy testing!

**Reference Git Repository**: [mcyt-api-supertest](https://github.com/MYCodeYatra/mcyt-api-supertest)
