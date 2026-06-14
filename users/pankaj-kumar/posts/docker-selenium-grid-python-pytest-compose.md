---
title: Docker Grid: Scaling Selenium Execution with Docker Compose
date: 28-Apr-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [python, selenium, ci-cd, docker, docker-compose, grid]
category: CI/CD Pipelines
categories: [CI/CD Pipelines, Python, Automation]
excerpt: >-
  Stop running tests one-by-one! Learn how to build an ephemeral Selenium Grid infrastructure using Docker Compose, allowing your CI/CD pipeline to scale browser execution instantly.
readTime: 6 min read
---

# Docker Grid: Scaling Selenium Execution with Docker Compose

Running 10 UI tests on your laptop takes 30 seconds. Running 500 UI tests on a single CI/CD server might take 30 minutes! When developers have to wait 30 minutes for a Pull Request pipeline to finish, they get frustrated and stop writing tests altogether.

To achieve fast CI/CD pipelines, we must execute our tests **in parallel**. However, if you try to open 10 Chrome windows simultaneously on a single Jenkins server, the server will run out of RAM and crash!

The enterprise solution to this problem is **Selenium Grid running inside Docker Containers**. In this article, we will teach you how to build a massive, ephemeral test infrastructure using Docker Compose.

---

## 1. What is Selenium Grid?

Selenium Grid is an architecture that consists of two components:
1. **The Hub (Router):** The master server that receives Pytest commands.
2. **The Nodes (Workers):** Independent machines (or containers) running Chrome, Firefox, or Edge. 

The Hub automatically routes your tests to the next available Node.

By placing this architecture inside Docker, we can instantly spin up (and tear down) perfectly clean browser environments without installing actual browsers on the host CI/CD server!

---

## 2. Creating the Docker Compose Grid

To create a Grid, we write a `docker-compose.yml` file. We will define one Hub container, one Chrome Node container, and one Firefox Node container.

**docker-compose.yml**

```yaml
version: '3'
services:
  # The Master Hub
  selenium-hub:
    image: selenium/hub:latest
    container_name: selenium-hub
    ports:
      - "4444:4444" # Expose the Hub port to our Pytest script
  # Chrome Worker Node
  chrome:
    image: selenium/node-chrome:latest
    shm_size: 2gb # Prevent Chrome from crashing due to memory limits
    depends_on:
      - selenium-hub
    environment:
      - SE_EVENT_BUS_HOST=selenium-hub
      - SE_EVENT_BUS_PUBLISH_PORT=4442
      - SE_EVENT_BUS_SUBSCRIBE_PORT=4443
  # Firefox Worker Node
  firefox:
    image: selenium/node-firefox:latest
    shm_size: 2gb
    depends_on:
      - selenium-hub
    environment:
      - SE_EVENT_BUS_HOST=selenium-hub
      - SE_EVENT_BUS_PUBLISH_PORT=4442
      - SE_EVENT_BUS_SUBSCRIBE_PORT=4443
```

To spin up this massive infrastructure, run a single command in your terminal:

```bash
docker-compose up -d
```
You now have a fully functioning Selenium Grid running on `http://localhost:4444`!

---

## 3. Configuring Pytest for Remote Execution

Now, we must update our Pytest `conftest.py` file. Instead of launching a local Chrome browser using `webdriver.Chrome()`, we will use `webdriver.Remote()` to send our tests to the Docker Hub!

**conftest.py**

```python
import pytest
from selenium import webdriver
from selenium.webdriver.chrome.options import Options
@pytest.fixture(scope="function")
def driver():
    # 1. Define Chrome Options
    options = Options()
    # 2. Point Selenium to the Docker Hub!
    grid_url = "http://localhost:4444/wd/hub"
    # 3. Launch the Remote Browser
    print(f"\n[Grid] Sending request to Docker Hub: {grid_url}")
    driver_instance = webdriver.Remote(
        command_executor=grid_url,
        options=options
    )
    yield driver_instance
    driver_instance.quit()
```

---

## 4. Scaling the Infrastructure

What if 1 Chrome Node isn't enough? What if you want to run 5 Chrome tests simultaneously? 

With Docker Compose, you do not need to rewrite your YAML file. You simply use the `--scale` command to dynamically create more containers!

```bash
# Spin up the Hub, 5 Chrome Containers, and 2 Firefox Containers!
docker-compose up -d --scale chrome=5 --scale firefox=2
```

Now, your Pytest script will send requests to the Hub, and the Hub will load-balance those requests across the 7 available containers!

## 5. Execution Output

Let's execute a basic test suite. The logs will indicate that the execution is happening remotely inside the Docker Network!

```bash
pytest tests/ -v -s
```

```text
============================= test session starts ==============================
collected 2 items
tests/test_ui.py::test_homepage 
[Grid] Sending request to Docker Hub: http://localhost:4444/wd/hub
PASSED
tests/test_login.py::test_auth 
[Grid] Sending request to Docker Hub: http://localhost:4444/wd/hub
PASSED
============================== 2 passed in 10.51s ==============================
```

Once the test suite is finished, you tear down the entire infrastructure with one command, ensuring your CI/CD server is perfectly clean for the next deployment:

```bash
docker-compose down
```

## Conclusion

Docker transforms complex infrastructure into simple code.
- A `docker-compose.yml` file allows you to define a Master Hub and multiple Browser Worker Nodes.
- Use `webdriver.Remote()` in Python to route your tests to the Hub.
- Use `--scale chrome=5` to instantly generate massive, parallel testing infrastructure.

But wait... if we have 5 Chrome containers, how do we actually tell Pytest to execute tests simultaneously? Right now, Pytest is still executing them one by one! 

In our next article, we will introduce **pytest-xdist**, the plugin that unleashes true parallel execution, dropping a 30-minute test suite down to 5 minutes!
