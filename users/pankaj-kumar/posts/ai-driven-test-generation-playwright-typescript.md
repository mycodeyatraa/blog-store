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
tags: ["playwright", "typescript", "ai", "codestart", "openai"]
category: AI in Testing
categories: ["Playwright TypeScript", "AI in Testing", "UI Automation"]
excerpt: >-
  Generate full end-to-end Playwright test suites using structured natural language prompts. Harness LLMs to write clean, modular Page Objects and assertions automatically.
readTime: 4 min read
---

# Prompt to Test: AI-Driven Test Generation in Playwright
 
Automating test generation is no longer a futuristic dream. With modern LLMs, we can translate raw product requirements directly into clean, executable Playwright TypeScript test cases.
 
---

## 1. Structured Prompts for Test Generation
 
To get high-quality code from an LLM, your prompt must contain:
1. The **Page HTML** (or DOM layout).
2. The **User Story** or requirement description.
3. The **Style Guide** (e.g., use Page Object Model, import Playwright test, use locator assertions).
 
---

## 2. Dynamic Playwright Code Generator
 
Here is how we programmatically request an LLM to generate a test:
 
```typescript
import { OpenAI } from 'openai';
import * as fs from 'fs';
const openai = new OpenAI();
async function generateTest(userStory: string, domLayout: string) {
  const systemPrompt = "You are an expert Playwright automation engineer. Output ONLY valid TypeScript code without markdown fences.";
  const response = await openai.chat.completions.create({
    model: 'gpt-4o',
    messages: [
      { role: 'system', content: systemPrompt },
      { role: 'user', content: `Create a Playwright test for this requirement: ${userStory}. DOM: ${domLayout}` }
    ]
  });
  const code = response.choices[0].message.content;
  fs.writeFileSync('tests/generated.spec.ts', code);
}
```
 
By integrating this generator into your workflow, you can accelerate framework scaffolding and coverage from day one.
