---
title: Orchestrating Playwright Monorepos with Turborepo
date: 24-May-2025
lastUpdated: 24-May-2025
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: ["playwright", "typescript", "monorepo", "turborepo", "workspaces", "ci-cd"]
category: Docker & Grid Scaling
categories: ["Docker & Grid Scaling", "UI Automation", "Playwright", "TypeScript", "Architecture"]
excerpt: >-
  Integrate Playwright into enterprise Monorepos by isolating tests via npm workspaces and guaranteeing execution order and remote caching using Turborepo.
readTime: 5 min read
---

As enterprise applications grow, engineering teams often adopt a **Monorepo** architecture. Instead of having separate repositories for the Frontend, Backend, UI Library, and E2E Tests, everything lives in a single repository.

While this makes code sharing easy, it introduces complexity for UI Automation. Playwright cannot test the frontend until the frontend is built! This is where **npm Workspaces** and **Turborepo** come in.

### 1. Organizing with npm Workspaces

npm Workspaces allow you to manage multiple distinct packages inside a single top-level `package.json`. 

In our architecture, the Playwright tests live in a dedicated `tests/e2e` workspace, completely isolated from the React frontend in `apps/web`.

**File:** `package.json` (Root)

```json
{
  "name": "enterprise-monorepo",
  "private": true,
  "workspaces": [
    "apps/*",
    "packages/*",
    "tests/*"
  ],
  "scripts": {
    "test:e2e": "npm run test --workspace=tests/e2e"
  }
}
```

If we execute `npm run test:e2e` from the root of the repository, npm automatically traverses down into the `tests/e2e` folder and executes the local Playwright suite.

### 2. Orchestration with Turborepo

While npm Workspaces manage the folder structure, they do not handle **Execution Order** or **Caching**. If we trigger our E2E tests, we must guarantee that the Frontend application is compiled first!

**Turborepo** is a high-performance build system designed specifically for Monorepos. It allows us to define dependencies between our npm scripts.

**File:** `turbo.json`

```json
{
  "$schema": "https://turbo.build/schema.json",
  "pipeline": {
    "build": {
      "dependsOn": ["^build"],
      "outputs": [".next/**", "dist/**"]
    },
    "test:e2e": {
      "dependsOn": ["build"],
      "outputs": ["playwright-report/**"]
    }
  }
}
```

### 3. The Power of "dependsOn"

Look closely at the `test:e2e` block in `turbo.json`. We explicitly stated: `"dependsOn": ["build"]`.

When a developer pushes code and the CI pipeline runs `npx turbo run test:e2e`, Turborepo will automatically intercept the command. It realizes that `test:e2e` depends on `build`, so it will first execute `npm run build` inside the `apps/web` frontend workspace. Only when the frontend successfully compiles will Turborepo execute the Playwright E2E suite!

Even better, if a developer only changed a Markdown file in a README and re-ran the pipeline, Turborepo's **Remote Caching** engine will realize the source code hasn't changed. It will instantly return the cached `playwright-report` from the previous run, turning a 10-minute CI pipeline into a 200-millisecond cache hit!

### Summary

By structuring your Playwright suite as an isolated **npm Workspace** and orchestrating it with **Turborepo**, you guarantee that your tests never execute against stale code, while simultaneously unlocking massive CI/CD performance gains through remote caching.
