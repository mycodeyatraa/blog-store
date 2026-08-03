---
title: Prompt to Test: AI-Driven Test Generation in Playwright
date: 02-Feb-2026
lastUpdated: 02-Feb-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: ["playwright", "typescript", "ai", "llm", "openai", "test-generation"]
category: AI in Testing
categories: ["Playwright TypeScript", "AI in Testing", "UI Automation"]
excerpt: >-
  Generate full end-to-end Playwright test suites using structured natural language prompts! Harness LLMs to scaffold modular Page Objects and assertions automatically.
readTime: 8 min read
---

# Prompt to Test: AI-Driven Test Generation in Playwright

Scaffolding UI test suites manually is repetitive and time-consuming. With advances in Large Language Models (LLMs), automation engineers can now convert raw user stories and DOM snapshots directly into production-ready **Playwright TypeScript** test specs.

---

## 1. Architecture of AI-Driven Test Generation

The automated generation pipeline combines three inputs:

1. **DOM Snapshot / Page Schema**: The clean structural HTML or ARIA tree of the target page.
2. **Acceptance Criteria**: Natural language description of expected behavior.
3. **Framework Style Guide**: Mandatory coding rules (e.g., use Page Objects, strict TypeScript types, `expect(locator)` assertions).

```
 +-------------------+
 | Acceptance Story  |  +-------------------+   +-------------------+   --> [ Prompt Builder Engine ] --> [ OpenAI GPT-4o ] --> [ Playwright Spec File ]
 | Page DOM Snapshot |  /
 +-------------------+ /
```

---

## 2. Implementing the AI Generator Script

Below is a complete Node.js / TypeScript utility that sends page context to OpenAI and saves the output spec:

**scripts/generate-playwright-test.ts**

```typescript
import { OpenAI } from 'openai';
import * as fs from 'fs';
import * as path from 'path';
 
const openai = new OpenAI({ apiKey: process.env.OPENAI_API_KEY });
 
interface TestGenRequest {
  featureName: string;
  userStory: string;
  domSnippet: string;
  outputPath: string;
}
 
export async function generatePlaywrightTest(req: TestGenRequest) {
  const systemPrompt = `
You are a Principal Automation Architect specializing in Playwright TypeScript.
Generate a complete, executable Playwright spec file based on the user requirement and DOM provided.
Rules:
1. Use '@playwright/test' framework syntax.
2. Use resilient locators (getByRole, getByLabel, getByTestId).
3. Include strict assertions using expect().
4. Output ONLY valid TypeScript code without markdown commentary.
`;
 
  const userPrompt = `
Feature: ${req.featureName}
Requirement: ${req.userStory}
DOM Layout:
${req.domSnippet}
`;
 
  const response = await openai.chat.completions.create({
    model: 'gpt-4o',
    messages: [
      { role: 'system', content: systemPrompt },
      { role: 'user', content: userPrompt }
    ],
    temperature: 0.2
  });
 
  const generatedCode = response.choices[0].message.content || '';
  fs.writeFileSync(req.outputPath, generatedCode, 'utf-8');
  console.log(`Successfully generated spec file: ${req.outputPath}`);
}
```

---

## 3. Generated Executable Playwright Spec

Here is an example of an AI-generated spec file produced by the generator:

**tests/generated-login.spec.ts**

```typescript
import { test, expect } from '@playwright/test';
 
test.describe('Generated User Login Suite', () => {
  test.beforeEach(async ({ page }) => {
    await page.goto('https://mycodeyatra.com/login');
  });
 
  test('should log in successfully with valid credentials', async ({ page }) => {
    // Fill credentials using resilient ARIA locators
    await page.getByLabel('Email Address').fill('user@example.com');
    await page.getByLabel('Password').fill('SecurePassword123');
    
    // Click submit
    await page.getByRole('button', { name: 'Sign In' }).click();
 
    // Assert dashboard redirection and heading
    await expect(page).toHaveURL('https://mycodeyatra.com/dashboard');
    await expect(page.getByRole('heading', { level: 1 })).toHaveText('Welcome, User');
  });
});
```

---

## Best Practices & Validation

1. **Sanitize Prompt Data**: Strip sensitive user tokens, keys, and PII from DOM snapshots before calling remote LLM APIs.
2. **Automated Linting**: Run `npx tsc --noEmit` and `npx eslint` on generated files immediately after synthesis.
3. **Human Review**: Always inspect generated tests before merging into primary production branches.
