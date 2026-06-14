---
title: Scaling Selenium: Dockerized Grids and Kubernetes Deployments
date: 25-Jun-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [python, selenium, docker, kubernetes, grid, gitlab-ci]
category: Cloud Grids & Advanced DevOps
categories: [Cloud Grids & Advanced DevOps, Python, Automation]
excerpt: >-
  Stop running tests sequentially! Learn how to use docker-compose to build a local Selenium Grid, scale infinitely using Kubernetes deployments, and trigger parallel execution from GitLab CI.
readTime: 6 min read
---

# Scaling Selenium: Dockerized Grids and Kubernetes Deployments

If you have 500 Python automation tests and each test takes 10 seconds to run, running them sequentially on your laptop will take **1.3 hours**. In a modern DevOps culture, waiting over an hour for feedback is unacceptable.

To reduce execution time to a few minutes, you must run your tests in **parallel**. But a single laptop cannot run 50 Chrome browsers simultaneously without crashing. You need a distributed cluster of machines.

In this article, we will teach you how to build an Enterprise Selenium Grid using **Docker**, and how to scale it infinitely using **Kubernetes (K8s)**!

---

## 1. The Selenium Grid Architecture

The Selenium Grid consists of two main components:
1. **The Hub (Router):** The central point that receives all test requests from your Pytest framework.
2. **The Nodes (Workers):** The actual machines/containers where the browsers live. The Hub forwards tests to available Nodes.

If you have 1 Hub and 10 Nodes, you can execute 10 Pytest functions at the exact same time!

---

## 2. Spinning Up a Grid with Docker Compose

Installing Java and downloading Selenium server `.jar` files manually is outdated. Today, the official Selenium team provides pre-configured Docker images.

Let's write a `docker-compose.yml` file to instantly spin up a Hub and 2 Chrome Nodes on any machine.

**docker-compose.yml**

```yaml
version: "3"
services:
  selenium-hub:
    image: selenium/hub:latest
    container_name: selenium-hub
    ports:
      - "4444:4444" # The port our Python script will connect to!
  chrome-node-1:
    image: selenium/node-chrome:latest
    depends_on:
      - selenium-hub
    environment:
      - SE_EVENT_BUS_HOST=selenium-hub
      - SE_EVENT_BUS_PUBLISH_PORT=4442
      - SE_EVENT_BUS_SUBSCRIBE_PORT=4443
  chrome-node-2:
    image: selenium/node-chrome:latest
    depends_on:
      - selenium-hub
    environment:
      - SE_EVENT_BUS_HOST=selenium-hub
      - SE_EVENT_BUS_PUBLISH_PORT=4442
      - SE_EVENT_BUS_SUBSCRIBE_PORT=4443
```

Run this command in your terminal:

```bash
docker-compose up -d
```
Boom! You now have a distributed Selenium Grid running locally. You can view the Hub's visual dashboard at `http://localhost:4444/grid/console`.

---

## 3. Connecting Pytest to the Docker Grid

Now that our Grid is running, we must tell our Python script to *stop* opening Chrome locally and *start* requesting a browser from the Remote Hub.

We do this using `webdriver.Remote()`.

**tests/conftest.py**

```python
import pytest
from selenium import webdriver
@pytest.fixture(scope="function")
def driver():
    # 1. Define the capabilities we want (We want Chrome)
    options = webdriver.ChromeOptions()
    # 2. Point Python to the Docker Hub!
    grid_url = "http://localhost:4444/wd/hub"
    print(f"\n[Grid] Requesting a Chrome container from {grid_url}...")
    # 3. Request the remote browser
    remote_driver = webdriver.Remote(
        command_executor=grid_url,
        options=options
    )
    yield remote_driver
    remote_driver.quit()
```

To run your tests in parallel, install `pytest-xdist`:

```bash
pip install pytest-xdist
```

Now, fire 2 tests simultaneously at the Grid:

```bash
pytest tests/ -n 2
```
Your two Pytest workers will connect to the Hub. The Hub will instantly route Test 1 to `chrome-node-1` and Test 2 to `chrome-node-2`!

---

## 4. Infinite Scaling with Kubernetes (K8s)

Docker Compose is great for a single server, but what if you need 100 browsers? A single server will run out of RAM. You need a cluster of servers, and for that, we use **Kubernetes**.

Kubernetes allows you to define a `Deployment` that dynamically creates and destroys Selenium Node containers across multiple physical AWS/Azure servers based on demand!

**kubernetes-chrome-deployment.yaml**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: selenium-node-chrome
spec:
  replicas: 10 # 🚀 KUBERNETES WILL MAINTAIN EXACTLY 10 CHROME NODES ACROSS YOUR CLUSTER!
  selector:
    matchLabels:
      app: selenium-node-chrome
  template:
    metadata:
      labels:
        app: selenium-node-chrome
    spec:
      containers:
      - name: selenium-node-chrome
        image: selenium/node-chrome:latest
        env:
          - name: SE_EVENT_BUS_HOST
            value: "selenium-hub-service"
          - name: SE_EVENT_BUS_PUBLISH_PORT
            value: "4442"
          - name: SE_EVENT_BUS_SUBSCRIBE_PORT
            value: "4443"
```

If a physical server in your cluster crashes, Kubernetes will instantly notice that 4 Chrome Nodes died, and it will automatically recreate those 4 Chrome Nodes on a healthy server. Your test automation suite will never experience downtime!

---

## 5. Integrating with GitLab CI & Azure DevOps

Once your Kubernetes Grid is live, you can integrate it directly into your Enterprise CI/CD pipelines.

When a developer merges code, GitLab CI or Azure DevOps will boot up a Pytest worker, which will fire tests at your massive Kubernetes Selenium Hub!

**Example .gitlab-ci.yml**

```yaml
test_e2e:
  stage: test
  image: python:3.10
  script:
    - pip install pytest selenium pytest-xdist
    # Point the tests to the internal Kubernetes Hub LoadBalancer IP!
    - export HUB_URL="http://selenium-hub.k8s.internal:4444/wd/hub"
    - pytest tests/ -n 10
```

## Conclusion

Local execution is for writing tests; Cloud execution is for running tests.
- Stop downloading `.jar` files; use **Docker Compose** to spin up local Selenium Grids instantly.
- Use `webdriver.Remote()` to configure Pytest to point at the Hub.
- Use `pytest-xdist` to execute tests in parallel.
- When your team needs massive scale, use **Kubernetes** to deploy your Selenium Nodes across an elastic cluster of servers.
- Trigger your Pytest suite from **GitLab CI** or **Azure DevOps** for true continuous testing!

In our next article, we will look at an alternative to maintaining your own Kubernetes infrastructure: **Outsourcing to Cloud Device Farms** like BrowserStack, Sauce Labs, and LambdaTest!
