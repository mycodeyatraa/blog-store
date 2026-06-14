---
title: Jenkins Integration: Executing Selenium Python with a Declarative Pipeline
date: 25-Apr-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [python, selenium, ci-cd, jenkins, jenkinsfile, groovy]
category: CI/CD Pipelines
categories: [CI/CD Pipelines, Python, Automation]
excerpt: >-
  Conquer enterprise CI/CD! Learn how to write a Groovy Jenkinsfile to build a Declarative Pipeline that automatically executes your headless Pytest Selenium suite and publishes JUnit XML reports.
readTime: 6 min read
---

# Jenkins Integration: Executing Selenium Python with a Declarative Pipeline

While GitHub Actions is modern and beautiful, the reality of Enterprise software is that Jenkins still rules the world. If you get hired at a Fortune 500 company, you will almost certainly be using Jenkins for your CI/CD pipelines.

In our previous article, we used YAML to configure our pipeline. Jenkins uses a different language entirely: **Groovy**. 

In this article, we will translate our DevSecOps workflow into a Declarative `Jenkinsfile` so you can automatically execute your Pytest suite on any corporate Jenkins server!

---

## 1. What is a Jenkinsfile?

Historically, QA Engineers would log into the Jenkins UI and manually configure their jobs by clicking through hundreds of dropdown menus. This is called "Freestyle" and it is a terrible practice because the configuration cannot be tracked in Git.

Today, we write a `Jenkinsfile` (with no file extension). This file lives in the root directory of your project right next to your `pytest.ini`. When Jenkins sees this file, it automatically builds the pipeline based on your code! This is known as **Pipeline as Code**.

---

## 2. Writing the Declarative Jenkinsfile

Our pipeline needs to do three things:
1. Setup a Python environment.
2. Install dependencies (Pytest, Selenium).
3. Execute the headless tests and generate a JUnit XML report.

**Jenkinsfile**

```groovy
pipeline {
    // Run this pipeline on any available Jenkins agent
    agent any 
    // Define environment variables (like secret tokens!)
    environment {
        PYTHONUNBUFFERED = '1'
        // Securely fetch credentials from Jenkins Credentials Manager
        API_TOKEN = credentials('mycodeyatra-api-token') 
    }
    stages {
        stage('Checkout Code') {
            steps {
                // Jenkins automatically pulls the latest code from GitHub
                checkout scm 
            }
        }
        stage('Install Dependencies') {
            steps {
                echo 'Installing Python Dependencies...'
                // Using sh to run Linux shell commands
                sh '''
                    python3 -m venv venv
                    . venv/bin/activate
                    pip install --upgrade pip
                    pip install -r requirements.txt
                '''
            }
        }
        stage('Execute Pytest Suite') {
            steps {
                echo 'Running Headless Selenium Suite...'
                // Run Pytest and output the results to an XML file!
                sh '''
                    . venv/bin/activate
                    pytest tests/ -v -s --junitxml=target/reports/test-results.xml
                '''
            }
        }
    }
    // This block always runs at the very end, even if the tests fail!
    post {
        always {
            echo 'Archiving Test Reports...'
            // Tell Jenkins to read the XML file and create a beautiful UI graph
            junit 'target/reports/*.xml'
        }
        success {
            echo '✅ Pipeline Passed! Ready for deployment.'
        }
        failure {
            echo '❌ Pipeline Failed! Blocking deployment.'
            // Optional: Send a Slack message here!
        }
    }
}
```

---

## 3. Handling Headless execution

Just like GitHub Actions, the Jenkins server executing this script does not have a monitor attached to it. It is a headless Linux box.

You **must** ensure your Python script launches Chrome in headless mode. If you forget `--headless=new`, the `Execute Pytest Suite` stage will crash with a cryptic `WebDriverException: unknown error: Chrome failed to start: crashed`.

**tests/conftest.py**

```python
# Reminder: This is required for Jenkins!
chrome_options.add_argument("--headless=new")
chrome_options.add_argument("--no-sandbox")
chrome_options.add_argument("--disable-dev-shm-usage")
```

---

## 4. Execution Output

When you commit this `Jenkinsfile` and push it to your repository, Jenkins will automatically detect it and trigger a build. 

You can view the execution logs directly in the Jenkins "Console Output" screen:

```text
Started by GitHub push by mycodeyatra
[Pipeline] Start of Pipeline
[Pipeline] stage
[Pipeline] { (Checkout Code)
[Pipeline] checkout
Fetching changes from the remote Git repository...
[Pipeline] }
[Pipeline] stage
[Pipeline] { (Install Dependencies)
[Pipeline] sh
+ python3 -m venv venv
+ . venv/bin/activate
+ pip install -r requirements.txt
Successfully installed selenium-4.x pytest-8.x
[Pipeline] }
[Pipeline] stage
[Pipeline] { (Execute Pytest Suite)
[Pipeline] sh
+ . venv/bin/activate
+ pytest tests/ -v -s --junitxml=target/reports/test-results.xml
============================= test session starts ==============================
collected 2 items
tests/test_ui.py::test_homepage PASSED
tests/test_login.py::test_auth PASSED
============================== 2 passed in 8.12s ===============================
[Pipeline] }
[Pipeline] stage
[Pipeline] { (Declarative: Post Actions)
[Pipeline] junit
Recording test results
[Pipeline] echo
✅ Pipeline Passed! Ready for deployment.
[Pipeline] }
[Pipeline] End of Pipeline
Finished: SUCCESS
```

## Conclusion

Jenkins might feel older than GitHub Actions, but understanding how to write a Groovy `Jenkinsfile` is a mandatory skill for Senior Automation Engineers.
- Keep your pipeline logic in the code repository (`Jenkinsfile`), never in the Jenkins UI.
- Use `sh` blocks to execute standard Linux terminal commands (`pip install`, `pytest`).
- Use the `post` block to execute the `junit` plugin, which parses your Pytest XML reports into beautiful, interactive graphs on the Jenkins dashboard!

In our next article, we will tackle the biggest bottleneck in automation: Execution Time. We will learn how to build a **Docker Grid** to run our tests across multiple containers simultaneously!
