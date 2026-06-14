---
title: Mastering Mouse & Keyboard Actions in Selenium TypeScript
date: 15-Nov-2024
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [typescript, selenium, mouse-actions, keyboard, drag-and-drop]
category: Selenium TypeScript
categories: [Selenium TypeScript, UI Automation, Core Automation]
excerpt: >-
  Standard click() methods are not enough for complex interactions. Learn how to chain gestures like Hover, Right-Click, Drag-and-Drop, and Click-and-Hold using the Actions API.
readTime: 6 min read
---

# Mastering Mouse & Keyboard Actions in Selenium TypeScript

Have you ever needed to hover over a dropdown menu to reveal hidden links, or perform a custom right-click context menu? Standard `.click()` and `.sendKeys()` methods are not enough for complex interactions.

In Selenium 4, the **Actions API** was completely overhauled to provide low-level control over the Mouse and Keyboard. 

In this tutorial, we will learn how to chain complex user gestures like Hovering, Right-Clicking, Drag-and-Drop, and Click-and-Hold using TypeScript on **[practice.mycodeyatra.com/#/mouse-actions](https://practice.mycodeyatra.com/#/mouse-actions)**.

---

## 1. Initializing the Actions Class

To start building complex gestures, you must initialize the `actions` object from your WebDriver instance.

```typescript
// Initialize async actions
const actions = driver.actions({ async: true });
```

---

## 2. Hover and Right-Click Context Menus

Modern web apps use CSS `:hover` states or JavaScript `onMouseEnter` events to show tooltips or menus. To trigger these, use `.move()`.
To trigger a Right-Click, use `.contextClick()`.

```typescript
// 1. Hover over an element
const hoverTarget = await driver.findElement(By.css("[data-testid='hover-target']"));
await actions.move({ origin: hoverTarget }).perform();
// 2. Right Click an element
const contextZone = await driver.findElement(By.css("[data-testid='context-menu-zone']"));
await actions.contextClick(contextZone).perform();
```

*Crucial Step: You must call `.perform()` at the end of your action chain to execute it!*

---

## 3. Complex Gestures: Drag & Drop and Resizing

The Actions API allows you to chain multiple atomic movements together to create complex gestures like Click, Hold, Move, and Release.

```typescript
// Drag and Drop Helper
await actions.dragAndDrop(sourceElement, targetElement).perform();
// Custom Click-Hold-Drag-Release (e.g., resizing a box)
await actions.move({ origin: resizerHandle })
             .press()  // Click and hold
             .move({ x: 50, y: 50, origin: resizerHandle }) // Drag 50px diagonally
             .release() // Let go
             .perform();
```

---

## 4. Writing the Mouse Actions Test

Let's write a complete Jest test to execute all of these gestures!

Create `tests/mouse_actions.test.ts`:

```typescript
import { Builder, WebDriver, By, until } from "selenium-webdriver";
import "chromedriver";
describe("Core UI Automation - Mouse Actions", () => {
  let driver: WebDriver;
  beforeAll(async () => {
    driver = await new Builder().forBrowser("chrome").build();
    await driver.manage().window().maximize();
    await driver.manage().setTimeouts({ implicit: 5000 });
  });
  afterAll(async () => {
    if (driver) {
      await driver.quit();
    }
  });
  it("Should perform hover, right-click, and drag-and-drop actions", async () => {
    console.log("Navigating to Mouse Actions Page...");
    await driver.get("https://practice.mycodeyatra.com/#/mouse-actions");
    const actions = driver.actions({ async: true });
    // 1. Hover Action
    const hoverTarget = await driver.wait(
      until.elementLocated(By.css("[data-testid='hover-target']")), 
      5000
    );
    await actions.move({ origin: hoverTarget }).perform();
    console.log("Hovered over the target element.");
    await driver.sleep(500);
    // 2. Right-Click (Context Menu) Action
    const contextZone = await driver.findElement(By.css("[data-testid='context-menu-zone']"));
    await actions.contextClick(contextZone).perform();
    console.log("Performed Right-Click on Context Zone.");
    // Verify custom context menu appeared
    const contextMenu = await driver.wait(
      until.elementLocated(By.css("[data-testid='custom-context-menu']")), 
      5000
    );
    expect(await contextMenu.isDisplayed()).toBe(true);
    console.log("Verified custom context menu is visible.");
    // Click outside to close the menu
    await actions.click(driver.findElement(By.css("h2"))).perform();
    // 3. Drag and Drop Action
    const dragSource = await driver.findElement(By.css("[data-testid='drag-source']"));
    const dropTarget = await driver.findElement(By.css("[data-testid='drop-target']"));
    await actions.dragAndDrop(dragSource, dropTarget).perform();
    console.log("Performed Drag and Drop.");
    // 4. Click and Hold (Resize)
    const resizer = await driver.findElement(By.css("[data-testid='resizer-handle']"));
    await actions.move({ origin: resizer })
                 .press()
                 .move({ x: 50, y: 50, origin: resizer })
                 .release()
                 .perform();
    console.log("Performed click, hold, and drag to resize the box.");
  });
});
```

---

## 5. Test Execution Output

```text
> mcyt-sel-typescript@1.0.0 test
> jest
 PASS  tests/mouse_actions.test.ts (7.214 s)
  Core UI Automation - Mouse Actions
    √ Should perform hover, right-click, and drag-and-drop actions (4112 ms)
  console.log
    Navigating to Mouse Actions Page...
  console.log
    Hovered over the target element.
  console.log
    Performed Right-Click on Context Zone.
  console.log
    Verified custom context menu is visible.
  console.log
    Performed Drag and Drop.
  console.log
    Performed click, hold, and drag to resize the box.
Test Suites: 1 passed, 1 total
Tests:       1 passed, 1 total
Snapshots:   0 total
Time:        7.514 s
Ran all test suites.
```

## Conclusion

The Selenium `Actions` class provides you with pixel-perfect control over the user's cursor and keyboard. You can build any complex sequence of events, from multi-select highlighting to drawing on an HTML canvas!

In our next tutorial, we will shift focus from browser interactions to **Execution Strategy: Running Tests in Parallel!**
