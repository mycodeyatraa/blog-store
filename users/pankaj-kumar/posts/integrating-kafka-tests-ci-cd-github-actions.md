---
title: Integrating Kafka Tests in CI/CD
date: 15-Mar-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: ["kafka", "ci-cd", "github-actions", "devops"]
category: API Testing
categories: [API Testing, Kafka, Async, BDD]
excerpt: >-
  Complete the cycle! Learn how to run Zookeeper, Kafka, and your test suites automatically in a GitHub Actions workflow.
readTime: 5 min read
---

# Blog #15: Integrating Kafka Tests in CI/CD

To complete our masterclass, we will automate our event-driven tests in a **GitHub Actions CI/CD Pipeline**.

## 1. Defining GitHub Actions Workflow File
We define a service container inside our YAML file to run Zookeeper and Kafka:

```yaml
name: Event-Driven CI
on: [push]
jobs:
  test:
    runs-on: ubuntu-latest
    services:
      kafka:
        image: confluentinc/cp-kafka:7.5.0
        ports:
          - 29092:29092
        env:
          KAFKA_ZOOKEEPER_CONNECT: localhost:2181
          # ... other environment variables
```

This ensures that every push builds the environment and verifies all integration flows automatically!
