---
title: Bypassing the UI: Creating Authenticated Browser Sessions
date: 20-Dec-2024
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [typescript, selenium, api-testing, authentication, cookies, localstorage, bypass]
category: Selenium TypeScript
categories: [Selenium TypeScript, API Testing Integration]
excerpt: >-
  Combine your API Token logic with Selenium WebDriver to instantly inject cookies or local storage directly into the browser, completely bypassing the UI login screen.
readTime: 4 min read
---

# Bypassing the UI: Creating Authenticated Browser Sessions

Throughout Phase 6, we have learned how to fetch JWT tokens, cache them to disk, and manage multi-user sessions using Axios. 

Now, we reach the culmination of Hybrid UI/API testing: **The Authentication Injector.**

If your test suite has 500 UI tests, forcing the browser to type a username and password 500 times is a massive waste of resources. By leveraging the backend APIs to fetch a token and injecting it directly into the browser's storage, we can log the user in instantly!

---

## 1. The Strategy

Modern web applications typically store their session state in one of two places:
1. **HTTP Cookies**
2. **Local Storage (or Session Storage)**

In this tutorial, we will use our `AuthUtils` class to grab a token from the backend, and then use `driver.executeScript()` to inject it directly into the browser's Local Storage!

---

## 2. Writing the Injection Script

Create `tests/authenticated_browser.test.ts`:

```typescript
import { Builder, WebDriver, By, until } from "selenium-webdriver";
import { AuthUtils } from "./utils/AuthUtils";
describe("Phase 6 - Advanced Authentication: Browser Sessions", () => {
  let driver: WebDriver;
  beforeAll(async () => {
    driver = await new Builder().forBrowser("chrome").build();
    await driver.manage().window().maximize();
  });
  afterAll(async () => {
    if (driver) {
      await driver.quit();
    }
  });
  it("Should bypass UI login by injecting a JWT token into localStorage", async () => {
    console.log("[Auth] Fetching JWT via API...");
    // 1. Fetch token via backend API (takes ~50ms)
    const token = await AuthUtils.getAuthToken();
    // 2. Navigate to the base domain FIRST.
    // CRITICAL: Browsers block setting localStorage or cookies if you are not already on the domain.
    // If you try to set storage on 'data:,', it will fail!
    await driver.get("https://practice.mycodeyatra.com/");
    // 3. Inject the token into localStorage using JavaScript Execution
    console.log("[Auth] Injecting token into Browser Local Storage...");
    await driver.executeScript(`window.localStorage.setItem('auth_token', '${token}');`);
    // 4. Refresh the page to apply the injected state
    // The front-end application will wake up, read localStorage, and immediately render the Dashboard!
    await driver.navigate().refresh();
    // 5. Verify that we are logged in
    // Note: In a real app, you would explicitly wait for the user avatar or dashboard panel.
    const body = await driver.findElement(By.tagName("body"));
    expect(body).toBeDefined();
    console.log("[Auth] Successfully bypassed UI login and landed on dashboard.");
  });
});
```

### Key Considerations
1. **Domain Context:** You **cannot** set cookies or local storage for `mycodeyatra.com` if your browser is currently parked on `google.com` or a blank page. You must navigate to the target domain *before* executing the injection script.
2. **State Application:** Injecting the token doesn't magically change the DOM. You must trigger a page reload (`driver.navigate().refresh()`) so the React/Angular/Vue application initializes with the new token.

---

## 3. Test Execution Output

Run the script:

```bash
> ENV_NAME=qa jest tests/authenticated_browser.test.ts
```

Output:

```text
 PASS  tests/authenticated_browser.test.ts
  Phase 6 - Advanced Authentication: Browser Sessions
    √ Should bypass UI login by injecting a JWT token into localStorage (1821 ms)
  console.log
    [Auth] Fetching JWT via API...
    [Auth] Injecting token into Browser Local Storage...
    [Auth] Successfully bypassed UI login and landed on dashboard.
```

## Conclusion

By marrying backend REST API automation with Selenium WebDriver, you have unlocked the true power of an SDET.

Instead of waiting 10 seconds for the UI to type credentials, click submit, and wait for network responses, you can achieve an authenticated browser session in under **2 seconds**. This one technique will shave hours off your CI/CD pipelines!

This marks the official end of Phase 6! In **Phase 7**, we will dive into an entirely new realm: **Security and Header Testing**. Stay tuned!
