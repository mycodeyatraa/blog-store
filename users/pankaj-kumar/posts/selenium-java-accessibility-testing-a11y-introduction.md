---
title: Automation for Everyone: An Introduction to Accessibility Testing (a11y)
date: 07-Jul-2025
lastUpdated: 07-Jul-2025
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [java, selenium, accessibility, a11y, wcag, axe-core, test-automation]
category: Selenium Java
categories: [Selenium Java, Accessibility Testing]
excerpt: >-
  What is web accessibility, and why must we automate it? Learn the basics of WCAG compliance and understand the architecture of injecting Axe-Core into Selenium to find UI violations.
readTime: 6 min read
---

# Automation for Everyone: An Introduction to Accessibility Testing (a11y)

In modern software development, we spend thousands of hours ensuring our web applications function correctly, look beautiful, and scale globally. But if your application cannot be used by a person who is visually impaired, motor-impaired, or colorblind, your application is fundamentally broken.

Accessibility (often abbreviated as **a11y** because there are 11 letters between the 'a' and the 'y') is no longer just a "nice-to-have" feature. In many countries, web accessibility is legally mandated (e.g., ADA compliance in the US, EN 301 549 in Europe).

In this new tutorial series, we will learn how to weaponize Selenium Java to automatically scan your web application for Accessibility violations during your CI/CD pipeline!

---

## 1. What makes a Website "Accessible"?

The gold standard for web accessibility is the **WCAG (Web Content Accessibility Guidelines)**. These guidelines define rules for how HTML should be constructed so that Assistive Technologies (like Screen Readers) can understand the page.

Some basic examples of WCAG rules include:
- **Alt Text:** Every `<img>` tag must have an `alt="description"` attribute so blind users know what the image is.
- **Color Contrast:** The text color must contrast heavily against the background color so visually impaired users can read it.
- **Keyboard Navigation:** Every interactive element (buttons, links, forms) must be reachable by pressing the `Tab` key, without requiring a mouse.
- **Form Labels:** Every `<input>` must have an associated `<label>` so screen readers announce what the user is typing into.

---

## 2. The Limitations of Manual Accessibility Testing

If you open Google Chrome, you can actually run a manual accessibility test right now:
1. Open Chrome DevTools (F12).
2. Go to the **Lighthouse** tab.
3. Check the "Accessibility" box and click "Analyze page load".

Lighthouse will scan the DOM and give you a score from 0 to 100, highlighting any missing alt text or poor contrast ratios.

**The Problem:** Running Lighthouse manually on every single page, after every single code commit, is impossible. Furthermore, Lighthouse only scans the initial page load. It doesn't test the accessibility of a modal window that only appears *after* you click a specific button.

This is why we must automate it.

---

## 3. How do we Automate Accessibility in Selenium?

Selenium is functionally designed to drive a browser. It clicks buttons and types text. It is *not* inherently an accessibility scanner. 

If an image is missing an `alt` tag, Selenium's `driver.findElement(By.id("logo")).isDisplayed()` will return `true` because the image physically exists. Selenium doesn't care that a screen reader can't understand it.

To solve this, we must inject a **JavaScript Accessibility Engine** directly into the browser while Selenium is running.

The industry standard engine for this is **axe-core** by Deque Systems.

---

## 4. The Architecture of an Automated a11y Test

The integration of Selenium and axe-core works in a 3-step sequence:

1. **Navigate:** Selenium drives the browser to the target page (e.g., logging in, opening a modal, filling a form).
2. **Inject & Execute:** Selenium uses its `JavascriptExecutor` to inject the `axe.min.js` library directly into the browser's DOM, and commands it to run a scan.
3. **Analyze:** The axe-core engine recursively scans the DOM, checking every element against hundreds of WCAG rules. It then returns a JSON object back to Selenium containing a list of all "Violations".

Here is a conceptual diagram of the architecture:

![diagram_1](https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/posts/selenium-java-accessibility-testing-a11y-introduction/images/diagram_1.png)

---

## 5. What Can and Cannot Be Automated

Before we dive into the code in the next tutorial, it is crucial to understand the limitations of automated accessibility testing.

**What Axe-Core CAN automate (about 30-40% of WCAG rules):**
- Missing `alt` attributes on images
- Missing `<label>` tags on inputs
- Invalid ARIA attributes (e.g., `aria-hidden="true"` on a focusable button)
- Insufficient color contrast ratios
- Multiple `<h1>` tags on a single page

**What Axe-Core CANNOT automate (Requires Human Testing):**
- **Quality of Alt Text:** Axe can verify that an `alt` attribute exists, but it cannot verify if the text actually makes sense. If an image of a dog has `alt="picture of a car"`, Axe will pass the test, but a human will be confused.
- **Logical Tab Order:** Axe can verify elements are focusable, but only a human can verify if tabbing through a complex form follows a logical visual sequence.
- **Video Captions:** Axe cannot watch a video and verify if the closed captions are accurate.

## Conclusion

Automated accessibility testing is an incredibly powerful safety net that prevents massive compliance violations from leaking into production. 

In our next tutorial, we will write our very first a11y test by integrating the **axe-core** library directly into our Selenium Java framework!
