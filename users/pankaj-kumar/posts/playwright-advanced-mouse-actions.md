---
title: "Advanced Mouse Actions (Hover, Drag & Drop)"
date: "2025-03-04"
description: "Go beyond simple clicks. Learn how to simulate complex mouse gestures like hovering, right-clicking for context menus, and drag-and-drop operations."
tags: ["Playwright", "TypeScript", "Mouse", "Hover", "Drag and Drop"]
---

Welcome to Blog 20 of the **Playwright TypeScript Mastery Series**!

Clicking buttons and typing text will get you through 80% of an application. But modern web apps are highly interactive. Elements appear when you hover over them, custom menus open when you right-click, and items are reordered using drag-and-drop.

In Selenium, you had to use the complex `Actions` class for this. In Playwright, these are natively integrated into your `Locator`!

Let's test these three actions against our live practice site (`https://practice.mycodeyatra.com/#/mouse-actions`):

### 1. Hovering

If an application reveals a hidden dropdown menu only when the mouse is physically resting on top of an element, you can use `.hover()`.

```typescript
import { test, expect } from '@playwright/test';
test('Hovering over an element', async ({ page }) => {
  await page.goto('https://practice.mycodeyatra.com/#/mouse-actions');
  
  const hoverTarget = page.getByTestId('hover-target');
  
  // Simulate physically moving the mouse over the element
  await hoverTarget.hover();
});
```

### 2. Right-Clicking (Context Menus)

To right-click, you don't need a special method. You just pass an options object into your standard `.click()` method!

```typescript
test('Right-Clicking (Context Menu)', async ({ page }) => {
  await page.goto('https://practice.mycodeyatra.com/#/mouse-actions');
  
  const contextZone = page.getByTestId('context-menu-zone');
  
  // Pass the button option to .click()
  await contextZone.click({ button: 'right' });
  
  // Verify our custom context menu appeared
  const customMenu = page.getByTestId('custom-context-menu');
  await expect(customMenu).toBeVisible();
});
```

*Note: You can also double-click using `.dblclick()`!*

### 3. Drag and Drop

Perhaps the most difficult interaction in UI automation is dragging an element across the screen and dropping it into a specific zone. Playwright turns this complex orchestration of `mousedown`, `mousemove`, and `mouseup` events into a single line of code!

```typescript
test('Drag and Drop', async ({ page }) => {
  await page.goto('https://practice.mycodeyatra.com/#/mouse-actions');
  
  const source = page.getByTestId('drag-source');
  const target = page.getByTestId('drop-target');
  
  // Playwright natively handles the entire drag and drop sequence!
  await source.dragTo(target);
  
  // Verify the drop was successful
  await expect(target).toHaveText('Dropped Successfully!');
});
```

### Execution Output

When you run `npx playwright test tests/blog20_mouse_actions.spec.ts`:

```
Running 3 tests using 1 worker
 
Successfully hovered over the target!
  OK   1 tests/blog20_mouse_actions.spec.ts:4:7 > Blog 20: Advanced Mouse Actions > Hovering over an element (1.1s)
Successfully right-clicked and verified the context menu!
  OK   2 tests/blog20_mouse_actions.spec.ts:17:7 > Blog 20: Advanced Mouse Actions > Right-Clicking (Context Menu) (558ms)
Successfully dragged and dropped the element!
  OK   3 tests/blog20_mouse_actions.spec.ts:32:7 > Blog 20: Advanced Mouse Actions > Drag and Drop (625ms)
 
  3 passed (3.8s)
```

### Conclusion

You now have a complete arsenal of Form and Mouse Interactions at your disposal. You can automate virtually any user journey through a web application.

In **Blog 21**, we are going to dive into the core of Test Architecture: **The Playwright Test Runner and Annotations (`skip`, `only`, `fail`)**!
