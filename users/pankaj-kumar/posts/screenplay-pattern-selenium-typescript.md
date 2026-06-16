---
title: Beyond POM: The Screenplay Pattern in TypeScript
date: 23-Nov-2024
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [typescript, selenium, screenplay-pattern, bdd, architecture]
category: Selenium TypeScript
categories: [Selenium TypeScript, Architecture & Patterns]
excerpt: >-
  Page Objects can quickly turn into unmaintainable God Objects. Learn the ultimate BDD framework architecture: The Screenplay Pattern.
readTime: 6 min read
---

# Beyond POM: The Screenplay Pattern in TypeScript

For years, the Page Object Model (POM) has been the undisputed king of UI automation architecture. However, POM has a fatal flaw: **Large classes violate the Single Responsibility Principle.**

As an application grows, a `HomePage.ts` class might accumulate 50 locators and 100 methods, becoming an unmaintainable "God Object".

To solve this, the industry introduced the **Screenplay Pattern** (popularized by Serenity BDD). It completely shifts the paradigm. Instead of organizing code by *Pages*, we organize code by *Actors, Tasks, and Abilities*.

---

## 1. The Screenplay Concept

In a Screenplay, we have:
* **Actors:** The users interacting with the system (e.g., "James the QA Engineer").
* **Abilities:** What the actor can do (e.g., `abilityToBrowse()` via WebDriver).
* **Tasks:** A specific goal the actor wants to achieve (e.g., `FillOutForm`).
* **Questions:** How the actor verifies the state of the UI (Assertions).

Let's build a minimalist TypeScript implementation.

---

## 2. Defining the Actor and Task Interfaces

Create `tests/screenplay/Actor.ts`. Notice how the `attemptsTo` method elegantly accepts a list of Tasks to perform in sequence!

```typescript
import { WebDriver } from "selenium-webdriver";
export interface Task {
  performAs(actor: Actor): Promise<void>;
}
export class Actor {
  constructor(
    public readonly name: string,
    public readonly driver: WebDriver
  ) {}
  async attemptsTo(...tasks: Task[]): Promise<void> {
    for (const task of tasks) {
      console.log(`${this.name} is attempting to perform a task...`);
      await task.performAs(this);
    }
  }
  abilityToBrowse(): WebDriver {
    return this.driver;
  }
}
```

---

## 3. Implementing the Tasks

Instead of giant Page Objects, we create tiny, single-purpose Task classes.
Create `tests/screenplay/Tasks.ts`:

```typescript
import { By, until } from "selenium-webdriver";
import { Actor, Task } from "./Actor";
export class NavigateToPracticeSite implements Task {
  async performAs(actor: Actor): Promise<void> {
    const driver = actor.abilityToBrowse();
    await driver.get("https://practice.mycodeyatra.com/#/form-practice");
  }
}
export class FillOutForm implements Task {
  constructor(private name: string, private email: string) {}
  // Syntactic sugar for extreme BDD readability
  static withDetails(name: string, email: string): FillOutForm {
    return new FillOutForm(name, email);
  }
  async performAs(actor: Actor): Promise<void> {
    const driver = actor.abilityToBrowse();
    const nameInput = await driver.wait(until.elementLocated(By.css("[data-testid='first-name']")), 5000);
    await nameInput.sendKeys(this.name);
    const emailInput = await driver.findElement(By.css("[data-testid='email']"));
    await emailInput.sendKeys(this.email);
  }
}
export class SubmitForm implements Task {
  async performAs(actor: Actor): Promise<void> {
    const driver = actor.abilityToBrowse();
    const submitBtn = await driver.findElement(By.css("[data-testid='submit-btn']"));
    await submitBtn.click();
  }
}
```

---

## 4. The Screenplay Test Script

When we piece it all together in our Jest test, the result is the most readable, maintainable, and declarative code imaginable. It reads exactly like a business requirement!

```typescript
import { WebDriver, By, until } from "selenium-webdriver";
import { AdvancedDriverFactory } from "./utils/advancedDriverFactory";
import { Actor } from "./screenplay/Actor";
import { NavigateToPracticeSite, FillOutForm, SubmitForm } from "./screenplay/Tasks";
describe("Architecture Phase - Screenplay Pattern", () => {
  let driver: WebDriver;
  beforeAll(async () => {
    driver = await AdvancedDriverFactory.getDriver();
  });
  afterAll(async () => {
    if (driver) { await driver.quit(); }
  });
  it("Should orchestrate the test using BDD Actors and Tasks", async () => {
    // 1. Create our Actor
    const james = new Actor("James the QA Engineer", driver);
    // 2. The Actor attempts a sequence of Tasks! (Pure BDD readability)
    await james.attemptsTo(
      new NavigateToPracticeSite(),
      FillOutForm.withDetails("James Bond", "007@mi6.gov"),
      new SubmitForm()
    );
    // 3. Questions (Assertions)
    console.log("James is verifying the result...");
    const resultMsg = await driver.wait(until.elementLocated(By.css("[data-testid='result-message']")), 5000);
    expect(await resultMsg.getText()).toContain("Form submitted successfully");
    console.log("Mission Accomplished!");
  });
});
```

---

## 5. Test Execution Output

```text
> mcyt-sel-typescript@1.0.0 test
> jest tests/screenplay_pattern.test.ts
 PASS  tests/screenplay_pattern.test.ts (6.291 s)
  Architecture Phase - Screenplay Pattern
    √ Should orchestrate the test using BDD Actors and Tasks (2213 ms)
  console.log
    James the QA Engineer is attempting to perform a task...
  console.log
    James the QA Engineer is attempting to perform a task...
  console.log
    James the QA Engineer is attempting to perform a task...
  console.log
    James is verifying the result...
  console.log
    Mission Accomplished!
```

## The End of the Journey

If you've followed this series from Blog 1 all the way to this final Screenplay Pattern, you have officially mastered **TypeScript UI Automation**. 

You've learned everything from resolving basic StaleElementExceptions, to intercepting CDP Network Traffic, to designing Enterprise-scale architectural patterns. 

Thank you for joining me on this incredible journey. Keep automating, keep innovating, and I'll see you in the next framework! 🚀
