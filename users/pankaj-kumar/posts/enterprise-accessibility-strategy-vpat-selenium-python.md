---
title: The Master Plan: Enterprise Accessibility Strategy in QA
date: 01-Mar-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [python, selenium, accessibility, strategy, vpat, enterprise, wcag]
category: Selenium Python
categories: [Selenium Python, Accessibility]
excerpt: >-
  The capstone of Phase 9. Learn how to implement the Accessibility Maturity Model, manage legacy violations with baselines, and understand the importance of VPATs in Enterprise B2B software.
readTime: 4 min read
---

# The Master Plan: Enterprise Accessibility Strategy in QA

Over the last six tutorials, we have learned how to automate Web Content Accessibility Guidelines (WCAG) using Axe-Core, validate dynamic ARIA states via Selenium, calculate mathematical color contrast ratios, and generate beautiful Allure reports.

However, technical implementation is only 20% of the battle. The other 80% is **Culture and Strategy**. 

If you just drop 500 accessibility failing tests into your enterprise CI/CD pipeline tomorrow, your engineering team will ignore them, disable the tests, and likely be very angry with you.

In this capstone tutorial for Phase 9, we will outline exactly how to roll out a sustainable, long-term Accessibility Strategy across a massive enterprise organization.

---

## 1. The Accessibility Maturity Model

You cannot become legally compliant overnight. Your organization must progress through the Accessibility Maturity Model:

### Level 1: Reactive (The Baseline)
- No automated checks.
- Accessibility is only considered when a customer complains or a lawsuit is filed.
- *Action:* Introduce Axe-Core locally. Start generating offline reports to show leadership the technical debt.

### Level 2: Proactive (The Baseline Shift)
- Axe-Core is integrated into the CI/CD pipeline, but it is **non-blocking** (tests run, generate warnings, but do not fail the build).
- Engineers start learning about WCAG organically by seeing the warnings.
- *Action:* Establish a "No New Violations" policy. 

### Level 3: Integrated (The Goal)
- Tests are **blocking**. If a PR introduces an accessibility violation, it is rejected.
- Product managers include accessibility criteria in Jira tickets.
- VPATs (Voluntary Product Accessibility Templates) are generated automatically.

---

## 2. Implementing the "No New Violations" Strategy

The biggest mistake QA engineers make is trying to fix the past. If you have a legacy application with 1,000 WCAG violations, you cannot fix them all today.

Instead, implement a **Baseline Strategy**:
1. Run Axe-Core on your entire application today.
2. Save the output to a `known_violations.json` file.
3. Update your test framework to *ignore* these known violations.
4. If a developer's Pull Request introduces a *new* violation that isn't in the baseline, the test fails!

Here is how you implement a baseline filter in Python:

```python
import json
from axe_selenium_python import Axe
from selenium import webdriver
# Load the historical baseline
with open("known_violations.json", "r") as f:
    known_violations = json.load(f)
known_ids = [v["id"] for v in known_violations]
driver = webdriver.Chrome()
driver.get("https://practice.mycodeyatra.com")
axe = Axe(driver)
axe.inject()
results = axe.run()
# Filter out legacy violations
new_violations = []
for v in results["violations"]:
    if v["id"] not in known_ids:
        new_violations.append(v)
if new_violations:
    print(f"❌ Pull Request rejected! You introduced {len(new_violations)} NEW accessibility violations.")
    # Fail CI/CD build here
else:
    print("✅ No new violations introduced!")
driver.quit()
```

This strategy immediately stops the bleeding without punishing developers for code they didn't write.

---

## 3. The 80/20 Rule of Automated A11y

Automated tools like Axe-Core are incredible, but you must set expectations with leadership: **Automation can only catch about 30% to 50% of all WCAG violations.**

For example:
- **Automation CAN check:** Does this `<img>` tag have an `alt` attribute?
- **Automation CANNOT check:** Is the text inside the `alt` attribute actually descriptive and useful to a blind person, or did the developer just write `alt="image"`?

### The Manual Testing Complement
To achieve true compliance (like WCAG 2.1 AA), your automated Selenium Python tests must be paired with:
1. **Keyboard Navigation Audits:** Manually unplugging the mouse and attempting to use the app with only the `Tab`, `Enter`, and `Space` keys.
2. **Screen Reader Audits:** Using NVDA (Windows) or VoiceOver (Mac) to manually navigate core user flows.
3. **User Testing:** Hiring actual users with disabilities to test your software.

---

## 4. What is a VPAT?

If you are selling B2B software, particularly to government agencies or universities, they will demand a **VPAT (Voluntary Product Accessibility Template)**.

A VPAT is a legal document that explains exactly how your product meets (or fails to meet) specific WCAG guidelines. 

As a QA Engineer, your automated HTML Reports (which we built in Blog 5) serve as the foundational data required by your Compliance team to fill out the technical sections of the VPAT. By running these scans on every release, you ensure the VPAT is never out of date.

## Conclusion: Phase 9 Complete!

Congratulations! You have mastered Accessibility Testing in Selenium Python. You now possess the rare ability to not only automate functional tests, but to ensure that the internet remains an inclusive space for all users, regardless of their physical abilities.

We are now ready to enter **Phase 10: Behavior Driven Development (BDD)**, where we will completely transform our Python code into readable English using the Gherkin syntax and `pytest-bdd`!
