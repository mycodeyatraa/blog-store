---
title: The Enterprise Standard: A Jenkins Pipeline Overview
date: 25-Jan-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [typescript, selenium, ci-cd, jenkins, groovy, devops]
category: Selenium TypeScript
categories: [Selenium TypeScript, CI/CD]
excerpt: >-
  Prepare for enterprise environments by learning how to translate your GitHub Actions YAML workflows into Groovy-based declarative Jenkinsfiles.
readTime: 4 min read
---

# The Enterprise Standard: A Jenkins Pipeline Overview

For the duration of this CI/CD phase, we have focused exclusively on GitHub Actions. It is cloud-native, uses simple YAML syntax, and requires zero server maintenance.

However, if you walk into a Fortune 500 bank, a healthcare provider, or a massive telecom company, you are very unlikely to find GitHub Actions. Instead, you will find a grumpy old server running **Jenkins**.

Jenkins is the undisputed grandfather of Continuous Integration. It is open-source, infinitely customizable, and entirely self-hosted (meaning highly secure corporations don't have to send their code to Microsoft's cloud).

If you want to be a Senior QA Automation Engineer, you must know how to translate your GitHub Actions workflows into Jenkins.

---

## 1. YAML vs. Groovy

GitHub Actions uses `YAML` (`.yml`), which is a static markup language (like JSON or XML). It is easy to read, but it is not a true programming language. You cannot easily write complex `for` loops or `try/catch` blocks in YAML.

Jenkins, on the other hand, uses **Groovy** to define its pipelines. 

Groovy is a fully-fledged object-oriented programming language that runs on the Java Virtual Machine. This means a `Jenkinsfile` is an actual script, capable of performing highly complex mathematical logic, database queries, and conditional branching directly inside the pipeline definition.

---

## 2. The Jenkinsfile Structure

In GitHub Actions, you place a `workflow.yml` file in a `.github` directory.
In Jenkins, you place a `Jenkinsfile` (no file extension) in the root of your repository.

Let's translate our Selenium Headless test execution from GitHub Actions into a Declarative `Jenkinsfile`:

```groovy
pipeline {
    // 1. Where should this run? (Equivalent to runs-on: ubuntu-latest)
    agent { 
        docker { 
            image 'node:18-bullseye' 
            args '--shm-size=2gb'
        }
    }
    // 2. Define Environment Variables (Equivalent to env:)
    environment {
        // Fetch the secret from the Jenkins Credential Store
        ADMIN_PASSWORD = credentials('prod-admin-password')
    }
    // 3. Define the Steps (Equivalent to jobs/steps)
    stages {
        stage('Checkout') {
            steps {
                // Equivalent to actions/checkout
                checkout scm
            }
        }
        stage('Install Chrome & Dependencies') {
            steps {
                sh '''
                    apt-get update
                    apt-get install -y google-chrome-stable
                    npm ci
                '''
            }
        }
        stage('Run Selenium Tests') {
            steps {
                // Execute the tests using the injected environment variable
                sh 'npm run test:bdd:smoke'
            }
        }
    }
    // 4. Post-execution Actions (Equivalent to if: always() and upload-artifact)
    post {
        always {
            // Save the HTML reports so they can be viewed in the Jenkins UI
            archiveArtifacts artifacts: 'reports/**/*.html', allowEmptyArchive: true
            // Save failure screenshots
            archiveArtifacts artifacts: 'screenshots/**/*.png', allowEmptyArchive: true
        }
        success {
            echo "Tests passed! Ready for production deployment."
        }
        failure {
            // Equivalent to our Slack Notification step
            slackSend channel: '#qa-alerts', 
                      color: 'danger', 
                      message: "Selenium Build Failed: ${env.BUILD_URL}"
        }
    }
}
```

---

## 3. Why Jenkins is Still Relevant

Looking at the code above, it's clear that a `Jenkinsfile` is slightly more verbose than GitHub Actions YAML. So why do enterprises still use it?

1. **Total Control:** You own the Jenkins server. If you need a highly specific VPN configuration to access an internal testing database, you can configure it at the operating system level.
2. **The Plugin Ecosystem:** Jenkins has been around since 2011. There are over 1,800 community-built plugins available, allowing it to integrate with literally any obscure enterprise software your company might use.
3. **Advanced Logic:** Because Groovy is a programming language, you can write "Scripted Pipelines" that dynamically generate stages at runtime based on the contents of a database!

## Conclusion

Congratulations! You now understand the core differences between modern, cloud-native YAML workflows (GitHub Actions) and legacy, highly-customizable Groovy scripts (Jenkins).

And with that... **we have officially reached the end of the curriculum!**

Over the last 11 Phases, you have evolved from a beginner learning how to locate a button on a web page, to an architect capable of building Behavior-Driven TypeScript frameworks executing in parallel across headless Docker containers in the cloud.

You have mastered **Enterprise Selenium TypeScript.**
