---
title: Zero Test Failures: AI Self-Healing Locators in Playwright
date: 03-Feb-2026
lastUpdated: 03-Feb-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: ["playwright", "typescript", "ai", "self-healing", "locators"]
category: AI in Testing
categories: ["Playwright TypeScript", "AI in Testing", "UI Automation"]
excerpt: >-
  Say goodbye to flaky locators. Learn how to implement self-healing test runs by calling LLMs dynamically to resolve broken selectors during execution.
readTime: 4 min read
---

# Zero Test Failures: AI Self-Healing Locators in Playwright
 
Locators break when UI code changes. A self-healing framework detects locator failures, calls an LLM with the page snapshot, generates a corrected locator, and resumes execution seamlessly.
 
## 1. Self-Healing Wrapper in Playwright
 
We can construct a helper to intercept locator timeouts:
 
```typescript
import { Page, Locator } from '@playwright/test';
import { OpenAI } from 'openai';
async function getHealedLocator(page: Page, brokenSelector: string): Promise<Locator> {
  try {
    return page.locator(brokenSelector);
  } catch (error) {
    const html = await page.content();
    const openai = new OpenAI();
    const response = await openai.chat.completions.create({
      model: 'gpt-4o',
      messages: [{ role: 'user', content: `Find the correct selector in this HTML for broken selector: ${brokenSelector}. HTML: ${html}` }]
    });
    const healedSelector = response.choices[0].message.content.trim();
    return page.locator(healedSelector);
  }
}
```
 
This prevents transient locator changes from failing your CI execution pipelines.
