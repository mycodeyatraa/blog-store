---
title: The Master Blueprint: Reporting Architecture
date: 06-Feb-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [typescript, selenium, observability, architecture, pipeline, elk, datadog, devops]
category: Selenium TypeScript
categories: [Selenium TypeScript, Observability]
excerpt: >-
  Pulling it all together. Visualize the complete Enterprise QA pipeline—from execution layers to data aggregation layers, connecting Docker, GitHub Actions, ELK, and Datadog.
readTime: 4 min read
---

# The Master Blueprint: Reporting Architecture

Over the last 13 Phases, we have slowly built a massive, enterprise-grade Selenium TypeScript automation framework. 

We started with simple `.click()` commands on a local laptop. Today, we are orchestrating parallel Docker containers in GitHub Actions, streaming JSON logs to Elasticsearch, and generating dynamic Allure reports.

But how does it all fit together?

As a Senior QA Architect, you must be able to visualize and explain the entire system. This is your **Reporting Architecture Blueprint**.

---

## 1. The Execution Layer (The Engine)

The journey begins when a developer opens a Pull Request.

1. **Trigger:** GitHub Actions detects the Pull Request.
2. **Provisioning:** GitHub spins up a cloud VM (the Runner).
3. **Orchestration:** The Runner pulls our `docker-compose.yml` file and spins up:
   - A Selenium Grid Hub
   - 5 Chrome Nodes
   - 1 Node.js test executor
4. **Execution:** The Node.js container runs `cucumber-js`. It parses our Gherkin Feature files, triggers our TypeScript Step Definitions, and sends WebDriver commands to the Selenium Grid.

---

## 2. The Data Generation Layer (The Sensors)

As the tests execute, they generate three distinct streams of data:

1. **The Cucumber Formatter Stream:** 
   The `allure-cucumberjs` plugin listens to the test execution. Every time a step passes or fails, it generates raw JSON files and saves them to the `./allure-results` folder.
2. **The Winston Log Stream:** 
   Our custom Winston logger injects a Correlation ID into every `console.log`, formats it as JSON, and writes it to standard output.
3. **The Datadog Trace Stream:** 
   Our `dd-trace` library hooks into the Node.js runtime, constantly measuring CPU usage, memory consumption, and execution duration, streaming this data directly to the Datadog Cloud API via HTTP POST.

---

## 3. The Aggregation Layer (The Brain)

Now, the test execution finishes. We have raw data scattered everywhere.

1. **Allure Generation:** GitHub Actions runs `allure generate ./allure-results`. It pulls the `history/` folder from the *previous* pipeline run, combines it with the new JSON files, and builds a static HTML dashboard.
2. **Logstash Forwarding:** Docker's logging driver intercepts the Winston JSON output and forwards it to Logstash, which ingests the logs and indexes them into an Elasticsearch cluster.

---

## 4. The Visualization Layer (The Dashboards)

Finally, the data is presented to the stakeholders via our 3-Tier Observability Strategy:

1. **Tier 1 (Executive):** The CTO opens Grafana, which pulls pass/fail metrics from Elasticsearch and infrastructure health from Datadog, displaying a high-level Green/Red health indicator.
2. **Tier 2 (Engineering):** The QA Engineer clicks the link posted by GitHub Actions and opens the Allure HTML report hosted on GitHub Pages. They look at the screenshots to see exactly what failed.
3. **Tier 3 (Debugging):** The DevOps Engineer opens Kibana, pastes the failing test's Correlation ID, and reads the perfectly ordered Winston JSON logs to find the exact database query that caused the timeout.

---

## Conclusion of Phase 13

This blueprint is the culmination of everything you have learned. You now understand that Test Automation is not just about writing a script to click a button. It is about building a highly scalable, observable software system.

But even the best architecture is useless if the system itself is difficult to maintain. 

As your company grows from 100 tests to 10,000 tests, how do you prevent your Page Object Models from turning into an unmaintainable nightmare of duplicate code?

Welcome to **Phase 14: Design Patterns**. In the next phase, we will leave the infrastructure behind and return to the code, mastering Advanced Page Object Models, the Factory Pattern, and the Singleton Pattern!
