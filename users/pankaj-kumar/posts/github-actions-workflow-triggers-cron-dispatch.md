---
title: On Demand and On Time: GitHub Actions Triggers
date: 18-Jan-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [typescript, selenium, ci-cd, github-actions, triggers, workflow_dispatch, cron]
category: Selenium TypeScript
categories: [Selenium TypeScript, CI/CD]
excerpt: >-
  Optimize your CI/CD pipeline by mastering GitHub Actions workflow triggers, including cron schedules, manual workflow_dispatch buttons, and targeted path filters.
readTime: 4 min read
---

# On Demand and On Time: GitHub Actions Triggers

In our previous tutorial, we created a basic GitHub Actions workflow that automatically triggered whenever someone pushed code to the `main` branch. 

However, running your entire UI automation suite on *every single commit* is highly inefficient. If a developer only updates a `README.md` file, there is absolutely no reason to burn cloud compute minutes spinning up a Ubuntu server to run Selenium tests.

Furthermore, sometimes QA Engineers just want to run the suite manually, or schedule it to run in the middle of the night.

We accomplish all of this using **Workflow Triggers**.

---

## 1. Path Filters (Ignoring the Noise)

The most common trigger is `on: push`. But we can optimize this by telling GitHub Actions to *ignore* pushes that only modify specific files or folders (like documentation or markdown files).

```yaml
on:
  push:
    branches:
      - main
    paths-ignore:
      - 'docs/**'
      - '*.md'
      - 'registry.json'
```

Alternatively, we can tell GitHub to *only* run the workflow if specific critical folders are modified:

```yaml
on:
  push:
    branches:
      - main
    paths:
      - 'tests/bdd/features/**'
      - 'src/**'
```

---

## 2. Manual Triggers (`workflow_dispatch`)

Often, a QA Engineer wants to execute a test suite without pushing any code. Maybe they want to run a "Smoke Test" against a Staging environment before a release.

To enable a literal "Run Workflow" button inside the GitHub UI, we use the `workflow_dispatch` trigger.

Even better, we can define **Input Parameters** so the user can choose *which* environment to test!

```yaml
on:
  workflow_dispatch:
    inputs:
      environment:
        description: 'Environment to test (e.g. dev, qa, prod)'
        required: true
        default: 'qa'
        type: choice
        options:
          - dev
          - qa
          - prod
      tags:
        description: 'Cucumber tags to execute (e.g. @smoke, @regression)'
        required: true
        default: '@smoke'
```

When someone clicks the "Run" button in GitHub, a popup will appear asking them to select an environment from the dropdown menu and type in their desired Cucumber tags!

You can then pass those variables directly into your test run:

```yaml
    steps:
      - name: Run Tests
        run: npm run test:bdd -- --tags "${{ github.event.inputs.tags }}"
```

---

## 3. Scheduled Triggers (Cron Jobs)

UI Automation suites, especially full `@regression` suites containing thousands of scenarios, can take hours to run. You do not want these running during peak business hours. 

You want them running at 3:00 AM while the team is asleep, so the test reports are waiting in everyone's inbox by 8:00 AM.

We accomplish this using the `schedule` trigger, which accepts standard POSIX Cron syntax:

```yaml
on:
  schedule:
    # Runs at 03:00 UTC every single day
    - cron: '0 3 * * *'
```

> **Pro Tip for Cron Syntax:** 
> The syntax is `Minute Hour Day-of-Month Month Day-of-Week`. 
> Want to run it at 5:00 PM every Friday? Use `0 17 * * 5`. 

---

## 4. Combining Triggers

The true power of GitHub Actions is that you can stack these triggers on top of each other! You can have a single workflow that runs on a schedule, but *also* provides a manual button, but *also* runs when a Pull Request is opened!

```yaml
name: Enterprise Test Execution
on:
  pull_request:
    branches: [ "main" ]
  workflow_dispatch:
  schedule:
    - cron: '0 3 * * 1-5' # 3 AM, Monday through Friday
```

## Conclusion

Mastering the `on:` block transforms your CI pipeline from a rigid script into a dynamic, highly configurable execution engine. 

But there is still a massive missing piece to our puzzle: **Selenium itself.**

If GitHub provides us with a headless Ubuntu Linux VM, how do we install the Google Chrome browser? Where does the `chromedriver` executable come from? 

In our next tutorial, we will finally learn how to configure the GitHub Actions VM to actually run a graphical browser!
