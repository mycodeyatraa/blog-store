---
title: The First Rule of ARIA: Validating WAI-ARIA Attributes with Axe
date: 11-Jul-2026
lastUpdated: 11-Jul-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [java, selenium, accessibility, a11y, aria, axe-core, test-automation]
category: Selenium Java
categories: [Selenium Java, Accessibility Testing]
excerpt: >-
  Bad ARIA is worse than no ARIA. Learn how to catch invalid ARIA roles with Axe-Core, and write explicit Selenium assertions to validate dynamic aria-expanded state changes on interactive components.
readTime: 6 min read
---

# The First Rule of ARIA: Validating WAI-ARIA Attributes with Axe

When you run your first Axe-Core scan against a modern web application, there is a very high probability that the vast majority of the violations returned will be related to **ARIA** (Accessible Rich Internet Applications) attributes.

To fix these bugs, you must first understand what ARIA is, why developers abuse it, and how to assert it correctly in Selenium.

---

## 1. What is ARIA?

Native HTML elements come with accessibility built-in. A `<button>` is inherently focusable via the `Tab` key, and a screen reader will automatically announce "Button" when the user selects it.

However, modern UI frameworks (like React or Angular) often build custom components using generic `<div>` tags.

```html
<div class="custom-toggle-switch" onclick="toggle()">Enable Feature</div>
```

To a sighted user, CSS makes this `<div>` look like a toggle switch. But to a screen reader, it's just plain text. It is not focusable, and it has no interactive meaning.

**ARIA** is a set of HTML attributes designed to fix this. It tells the screen reader what the generic `<div>` is actually supposed to represent.

```html
<div role="switch" aria-checked="false" tabindex="0" onclick="toggle()">
  Enable Feature
</div>
```
Now, the screen reader correctly announces: *"Enable Feature, Switch, Not Checked"*.

---

## 2. The First Rule of ARIA

The first rule of ARIA is: **No ARIA is better than Bad ARIA.**

Developers often add ARIA attributes without understanding them, resulting in broken screen reader experiences.

For example, adding `aria-hidden="true"` to an element completely hides it from the screen reader. If a developer accidentally adds `aria-hidden="true"` to the main submit `<button>`, a blind user will literally have no way to submit the form!

---

## 3. Common ARIA Violations Checked by Axe

When you run `new AxeBuilder().analyze(driver)`, the Axe engine automatically validates the DOM against strict ARIA rules. Some of the most common failures include:

1. **`aria-roles`**: Ensures that every `role=""` attribute used is a valid W3C standard role (e.g., you cannot use `role="clickable-thing"`).
2. **`aria-valid-attr`**: Ensures that the specific `aria-*` attributes used actually exist in the specification.
3. **`aria-allowed-attr`**: Ensures that an ARIA attribute is allowed on the specific element. (e.g., `aria-checked` is allowed on `role="checkbox"`, but it is **illegal** on `role="link"`).
4. **`aria-required-children`**: Elements like `role="list"` **must** contain `role="listitem"` children. If they don't, Axe fails the test.

---

## 4. Validating Dynamic ARIA States in Selenium

While Axe-Core is fantastic for static DOM analysis, many ARIA attributes are highly dynamic. 

For example, when an accordion menu expands, its `aria-expanded` attribute must dynamically flip from `"false"` to `"true"`. You can write explicit Selenium Java assertions to validate these dynamic behavioral states!

```java
import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.chrome.ChromeDriver;
import org.testng.Assert;
import org.testng.annotations.Test;
public class AriaStateTest {
    @Test
    public void testAccordionAriaState() {
        WebDriver driver = new ChromeDriver();
        driver.get("https://practice.mycodeyatra.com/faq");
        // 1. Locate the Accordion Header
        WebElement faqHeader = driver.findElement(By.id("faq-question-1"));
        // 2. Assert Initial State (Should be collapsed)
        Assert.assertEquals(faqHeader.getAttribute("aria-expanded"), "false", 
            "Aria state should be false when accordion is closed!");
        // 3. Interact (Click to expand)
        faqHeader.click();
        // 4. Assert Dynamic State Change!
        Assert.assertEquals(faqHeader.getAttribute("aria-expanded"), "true", 
            "Aria state MUST update to true when accordion is expanded!");
        driver.quit();
    }
}
```

## Conclusion

Understanding ARIA is the most difficult part of Web Accessibility. By leveraging Axe-Core to catch invalid or illegal attributes, and writing explicit `getAttribute()` assertions to validate dynamic state changes, you can ensure that your custom UI components are flawlessly accessible to screen readers.

In our next tutorial, we will explore another critical pillar of WCAG compliance: **Color Contrast Testing**!
