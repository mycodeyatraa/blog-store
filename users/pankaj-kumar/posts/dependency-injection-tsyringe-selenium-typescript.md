---
title: Enterprise Scale: Dependency Injection in TypeScript
date: 29-Nov-2024
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [typescript, selenium, dependency-injection, tsyringe, design-patterns]
category: Selenium TypeScript
categories: [Selenium TypeScript, Framework Utilities & Configuration]
excerpt: >-
  Say goodbye to Constructor Hell. Learn how to use tsyringe and reflect-metadata to automatically inject Page Objects and Services.
readTime: 4 min read
---

# Enterprise Scale: Dependency Injection in TypeScript

As your framework scales, your Page Objects will start relying on other services. 
For example, `HomePage.ts` might need the `DatabaseService` to fetch user data, and the `DatabaseService` might need the `Logger` utility.

If you instantiate these manually, you end up with "Constructor Hell":

```typescript
const logger = new Logger();
const db = new DatabaseService(logger);
const homePage = new HomePage(db, logger);
// It gets worse the larger the framework grows!
```

**Dependency Injection (DI)** solves this by automatically providing classes with their required dependencies. We will use the Microsoft library `tsyringe`.

---

## 1. Setup

First, install `tsyringe` and `reflect-metadata` (required for TypeScript decorators):

```bash
npm install tsyringe reflect-metadata
```

You must also update your `tsconfig.json` to enable decorators:

```json
{
  "compilerOptions": {
    "experimentalDecorators": true,
    "emitDecoratorMetadata": true
  }
}
```

---

## 2. Using the `@injectable` Decorator

Instead of manually creating instances using `new`, we decorate our classes with `@injectable()`. This registers them in the global DI container.

Create `tests/dependency_injection.test.ts`:

```typescript
import "reflect-metadata"; // MUST be imported once at the top of the app!
import { injectable, container } from "tsyringe";
// 1. A mock Database Service that our tests might need to fetch data
@injectable()
export class DatabaseService {
  fetchUserCredentials(): { user: string, pass: string } {
    console.log("[DB] Connecting to DB and fetching users...");
    return { user: "admin_di", pass: "password_di" };
  }
}
// 2. A mock Page Object that depends on the Database Service
@injectable()
export class LoginPage {
  // We NEVER pass the db manually. tsyringe detects it and injects it!
  constructor(private db: DatabaseService) {}
  login() {
    const creds = this.db.fetchUserCredentials();
    console.log(`[UI] Logging in as ${creds.user} with ${creds.pass}`);
    return true;
  }
}
```

---

## 3. Resolving from the Container

Now look at how clean our test file is. We don't instantiate *anything* manually! We just ask the container to `resolve` the `LoginPage`.

```typescript
describe("Phase 4 - Dependency Injection", () => {
  it("Should automatically inject dependencies without the 'new' keyword", () => {
    // Instead of:
    // const db = new DatabaseService();
    // const page = new LoginPage(db);
    // We just do this!
    const loginPage = container.resolve(LoginPage);
    const result = loginPage.login();
    expect(result).toBeTruthy();
  });
});
```

---

## 4. Test Execution Output

```text
> mcyt-sel-typescript@1.0.0 test
> jest tests/dependency_injection.test.ts
 PASS  tests/dependency_injection.test.ts
  Phase 4 - Dependency Injection
    √ Should automatically inject dependencies without the 'new' keyword (4 ms)
  console.log
    [DB] Connecting to DB and fetching users...
  console.log
    [UI] Logging in as admin_di with password_di
```

## Conclusion

Dependency Injection drastically reduces tight coupling in your framework. If the `DatabaseService` changes its constructor tomorrow, your `LoginPage` doesn't care. The container handles the wiring automatically!

In the final tutorial of Phase 4, we will look at **Custom Jest Reporters** to format our execution data into beautiful dashboards!
