---
title: Dockerizing Playwright for Cloud Environments
date: 18-May-2025
lastUpdated: 18-May-2025
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: ["playwright", "typescript", "docker", "docker-compose", "containers", "ci-cd"]
category: Docker & Grid Scaling
categories: ["Docker & Grid Scaling", "UI Automation", "Playwright", "TypeScript", "DevOps"]
excerpt: >-
  Eliminate the 'works on my machine' paradox by containerizing your Playwright automation suite using the official Microsoft Docker image and Docker Compose volume mounts.
readTime: 4 min read
---

When your automation suite transitions from a local repository to a Cloud CI/CD pipeline, you run into the "It works on my machine" problem. A test might pass on your MacBook, but fail on the Ubuntu GitHub Actions runner because it is missing a specific lib-x11 OS dependency required by Chromium.

To guarantee your Playwright tests run exactly the same way in CI/CD as they do locally, you must **Dockerize** your automation suite.

### 1. The Official Playwright Image

Playwright requires specific operating system dependencies (like fonts and C++ libraries) to boot headless browsers. Rather than installing these manually via `apt-get`, Microsoft maintains an official Docker image (`mcr.microsoft.com/playwright`) that comes pre-packaged with Ubuntu, Node.js, Chromium, Firefox, WebKit, and every required OS dependency!

**File:** `Dockerfile`

```dockerfile
# Pull the official Playwright image (matches your Playwright version!)
FROM mcr.microsoft.com/playwright:v1.43.0-jammy
 
WORKDIR /app
 
# Copy package configurations and install node modules
COPY package*.json ./
RUN npm ci
 
# Copy your actual test scripts
COPY . .
 
# Set the default execution command
CMD ["npx", "playwright", "test"]
```

### 2. Running with Docker Compose

Running `docker build` and `docker run` manually with a bunch of environment variables is tedious. It's much easier to configure a `docker-compose.yml` file. 

Compose allows us to inject environment variables and, crucially, map a **Volume**. When the container finishes running the tests, the container shuts down and destroys its filesystem. If we don't map a Volume, we will lose our Playwright HTML report!

**File:** `docker-compose.yml`

```yaml
version: '3.8'
services:
  playwright-tests:
    build: .
    environment:
      - CI=true
      - BASE_URL=https://staging.app.com
    # Volume Mount: Extracts the report out of the container onto the Host machine!
    volumes:
      - ./playwright-report:/app/playwright-report
```

### 3. Execution

Now, running your entire test suite inside an isolated Linux container takes a single command:

```bash
docker-compose up --build
```

### Summary

By wrapping your Playwright suite in a Docker container using the official `mcr.microsoft.com/playwright` image, you achieve absolute environment parity. Whether the suite runs on your local Windows laptop, an AWS EC2 instance, or a Jenkins worker node, you are guaranteed that the browser binaries and OS dependencies are identical.
