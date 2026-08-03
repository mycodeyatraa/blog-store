---
title: Stunning Dashboards: Allure Reporting Basics in Python
date: 15-Feb-2026
lastUpdated: 15-Feb-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: ["selenium", "python", "allure", "reporting", "dashboards"]
category: Reporting and Observability
categories: ["Reporting and Observability", "Python", "Automation"]
excerpt: >-
  Upgrade your console output to stunning visual reports. Learn how to configure, generate, and view interactive Allure reports with pytest.
readTime: 4 min read
---

# Stunning Dashboards: Allure Reporting Basics in Python
 
Allure Framework is a flexible, lightweight multi-language test report tool designed to show clean execution dashboards.
 
---

## 1. Setting Up Allure in Pytest
 
Install the allure-pytest library:
 
```bash
pip install allure-pytest
```
 
---

## 2. Generating Reports
 
Run your tests and specify the results directory:
 
```bash
pytest --alluredir=allure-results
allure serve allure-results
```
 
This transforms text logs into interactive, executive-ready dashboard interfaces.
