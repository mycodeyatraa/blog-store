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
tags: ["playwright", "typescript", "ai", "self-healing", "locators", "resilience"]
category: AI in Testing
categories: ["Playwright TypeScript", "AI in Testing", "UI Automation"]
excerpt: >-
  Eliminate locator breakage caused by frontend DOM changes! Build AI self-healing locator wrappers in Playwright TypeScript using LLM fallback strategies.
readTime: 8 min read
---

# Zero Test Failures: AI Self-Healing Locators in Playwright

The single most frequent cause of automated UI test failure is modified DOM structure—when a developer changes an element ID, class name, or layout hierarchy.

By implementing **AI Self-Healing Locators**, test automation frameworks can catch locator failure exceptions in real time, query an LLM with the active DOM tree, repair the broken locator dynamically, and continue test execution without breaking CI runs.

---

## 1. Self-Healing Execution Flowchart

```
 +-----------------------+
 |  Execute Test Action  |
 +-----------------------+
             |
             v
 [ Element Found? ] ---> YES ---> Continue Test
             |
            NO (Timeout / NoSuchElement)
             |
             v
 +-----------------------------------+
 | 1. Capture Full Page DOM          |
 | 2. Send Broken Selector to LLM    |
 | 3. LLM returns Healed Selector    |
 | 4. Retry Action with New Selector |
 +-----------------------------------+
```

---

## 2. Self-Healing Playwright Wrapper Class

Below is a self-healing Playwright wrapper class written in TypeScript:

**utils/SelfHealingPage.ts**

```typescript
import { Page, Locator, expect } from '@playwright/test';
import { OpenAI } from 'openai';
 
export class SelfHealingPage {
  private openai: OpenAI;
 
  constructor(public page: Page) {
    this.openai = new OpenAI({ apiKey: process.env.OPENAI_API_KEY });
  }
 
  async clickHealed(selector: string, description: string): Promise<void> {
    try {
      // Attempt standard locator click with 3 second timeout
      await this.page.locator(selector).click({ timeout: 3000 });
    } catch (error) {
      console.warn(`[Self-Healing Triggered] Primary selector failed: '${selector}' for (${description})`);
      
      const healedSelector = await this.healSelector(selector, description);
      console.log(`[Self-Healing Success] Repaired selector: '${healedSelector}'`);
      
      // Retry action using healed selector
      await this.page.locator(healedSelector).click({ timeout: 5000 });
    }
  }
 
  private async healSelector(failedSelector: string, description: string): Promise<string> {
    // Capture clean snapshot of current body HTML
    const bodyHtml = await this.page.evaluate(() => document.body.innerHTML);
    const truncatedHtml = bodyHtml.substring(0, 15000); // Limit payload size
 
    const response = await this.openai.chat.completions.create({
      model: 'gpt-4o',
      messages: [
        {
          role: 'system',
          content: 'You are an AI locator repair engine. Analyze the DOM HTML and return ONLY a valid Playwright CSS or ARIA selector for the described element.'
        },
        {
          role: 'user',
          content: `Failed Selector: ${failedSelector}
Element Description: ${description}
Active DOM:
${truncatedHtml}`
        }
      ],
      temperature: 0.1
    });
 
    const healed = response.choices[0].message.content?.trim() || '';
    return healed.replace(/```/g, ''); // Strip potential code fences
  }
}
```

---

## 3. Writing Tests with Self-Healing Capabilities

**tests/test-self-healing.spec.ts**

```typescript
import { test, expect } from '@playwright/test';
import { SelfHealingPage } from '../utils/SelfHealingPage';
 
test('should complete checkout despite broken UI button selector', async ({ page }) => {
  await page.goto('https://mycodeyatra.com/checkout');
 
  const healingPage = new SelfHealingPage(page);
 
  // Even if '#old-submit-btn-id' was changed in frontend, AI heals it dynamically
  await healingPage.clickHealed('#old-submit-btn-id', 'Main Complete Purchase Button');
 
  await expect(page.getByText('Order Complete')).toBeVisible();
});
```

---

## Key Benefits

1. **Drastically Reduces Flakiness**: Prevents CI pipeline failures due to minor UI tweaks.
2. **Automated Defect Logging**: Automatically log healed locators into Jira or Slack so developers can update selectors permanently.
