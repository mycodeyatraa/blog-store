---
title: GitHub Actions: Automating Pytest Selenium on Every Pull Request
date: 22-Apr-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [python, selenium, ci-cd, github-actions, pytest, devops]
category: CI/CD Pipelines
categories: [CI/CD Pipelines, Python, Automation]
excerpt: >-
  Stop running tests locally! Learn how to build a true DevSecOps pipeline using GitHub Actions YAML to automatically execute your headless Selenium Python suite in the cloud on every Pull Request.
readTime: 6 min read
---

# GitHub Actions: Automating Pytest Selenium on Every Pull Request

Throughout this curriculum, we have built an incredible Python automation framework. We have Authentication bypasses, Security headers validation, API integration, and AI Visual testing.

But right now, the framework only runs when you type `pytest` on your local laptop. **This is not true automation.**

True Continuous Integration (CI) means that your tests run automatically on a remote server every time a developer attempts to merge code. If the tests fail, the developer is blocked from deploying their code.

In this article, we will establish true DevSecOps by integrating our framework into **GitHub Actions**!

---

## 1. What is GitHub Actions?

GitHub Actions is a CI/CD platform built directly into GitHub. It allows you to define "Workflows" using YAML files. 

When a specific event happens (like a developer opening a Pull Request), GitHub spins up a pristine Linux server in the cloud, installs your Python environment, installs Google Chrome, and executes your Pytest suite!

If the tests pass, a green checkmark appears on the Pull Request. If they fail, a red 'X' blocks the merge!

---

## 2. Preparing the Python Framework

Before we push our code to GitHub, we must ensure our Selenium tests are configured to run headlessly. Cloud servers do not have monitors, so attempting to open a visible UI window will instantly crash the test!

**conftest.py**

```python
import pytest
from selenium import webdriver
from selenium.webdriver.chrome.options import Options
@pytest.fixture(scope="session")
def driver():
    chrome_options = Options()
    # 1. Run Headless (CRITICAL for CI/CD)
    chrome_options.add_argument("--headless=new")
    # 2. Disable GPU and Sandbox (Required for Linux Servers)
    chrome_options.add_argument("--disable-gpu")
    chrome_options.add_argument("--no-sandbox")
    chrome_options.add_argument("--disable-dev-shm-usage")
    # 3. Set a standard resolution to ensure elements are visible
    chrome_options.add_argument("--window-size=1920,1080")
    driver_instance = webdriver.Chrome(options=chrome_options)
    yield driver_instance
    driver_instance.quit()
```

---

## 3. Creating the GitHub Actions YAML

GitHub Actions looks for YAML files inside a very specific directory in your repository: `.github/workflows/`.

Let's create our pipeline configuration file.

**.github/workflows/selenium-tests.yml**

```yaml
name: Selenium UI Automation Suite
# 1. Triggers: When should this workflow run?
on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]
# 2. Jobs: The actual work to perform
jobs:
  ui-tests:
    runs-on: ubuntu-latest # Spins up a pristine Ubuntu Linux server
    steps:
    # Step 1: Download our code from the repository
    - name: Checkout Code Repository
      uses: actions/checkout@v4
    # Step 2: Install Python 3.12
    - name: Set up Python
      uses: actions/setup-python@v5
      with:
        python-version: '3.12'
    # Step 3: Install our framework dependencies
    - name: Install Python Dependencies
      run: |
        python -m pip install --upgrade pip
        pip install pytest selenium requests python-dotenv
    # Step 4: Ensure the Ubuntu server has Chrome installed!
    - name: Install Google Chrome
      run: |
        sudo apt-get update
        sudo apt-get install -y google-chrome-stable
    # Step 5: Execute the Pytest Suite
    - name: Run Pytest Automation Suite
      run: |
        pytest tests/ -v -s --junitxml=report.xml
```

---

## 4. Execution Output

Once you commit this `.yml` file and push it to GitHub, navigate to the "Actions" tab in your repository. You will see a live console output of the cloud server booting up and running your tests!

```text
Run pytest tests/ -v -s --junitxml=report.xml
============================= test session starts ==============================
platform linux -- Python 3.12.0, pytest-8.0.0
rootdir: /home/runner/work/mycodeyatra/mycodeyatra
collected 2 items
tests/test_login.py::test_valid_login PASSED
tests/test_api_mock.py::test_mocked_network_call PASSED
============================== 2 passed in 12.45s ==============================
Job 'ui-tests' completed successfully.
```

Because the test passed, the Pull Request will show a beautiful green checkmark indicating that the "Selenium UI Automation Suite" is successful, and the developer is permitted to merge their code!

## Conclusion

Running tests locally is development. Running tests automatically in the cloud is Engineering.
- Always configure `ChromeOptions` with `--headless=new` and `--no-sandbox` to ensure Selenium doesn't crash on Linux servers.
- Use `.github/workflows/` to define your CI/CD pipelines in YAML.
- GitHub Actions automatically handles downloading Google Chrome and executing your Pytest suite on every single Pull Request!

While GitHub Actions is incredible for open-source and modern repos, many massive enterprises still use Jenkins. In our next article, we will teach you how to write a declarative `Jenkinsfile` to execute this exact same pipeline in a legacy Jenkins environment!
