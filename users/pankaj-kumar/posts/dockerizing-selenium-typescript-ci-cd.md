---
title: Escaping Dependency Hell: Docker & CI/CD Integration
date: 24-Jan-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [typescript, selenium, ci-cd, github-actions, docker, containers, devops]
category: Selenium TypeScript
categories: [Selenium TypeScript, CI/CD]
excerpt: >-
  Solve "It works on my machine" forever by packaging your Selenium TypeScript framework into an isolated, portable Docker container that runs flawlessly in any CI environment.
readTime: 4 min read
---

# Escaping Dependency Hell: Docker & CI/CD Integration

"But it works on my machine!"

This is the most famous phrase in software engineering. A developer writes code that works perfectly on their Macbook, but when they send it to a QA Engineer running Windows, it crashes. Or they deploy it to an Ubuntu Server, and it fails because the Node.js versions don't match.

This phenomenon is called "Dependency Hell." And the ultimate solution to Dependency Hell is **Docker**.

---

## 1. What is Docker?

Docker allows you to wrap your code, your libraries, your environment variables, and the *entire operating system* into a single, portable package called a **Container**.

If you build a Docker container on your Macbook, you can send that container to your teammate, and it will execute identically. You can send it to AWS, Azure, or GitHub Actions, and it will execute identically.

For Selenium TypeScript automation, Docker is revolutionary. You no longer have to worry about whether a server has Google Chrome installed, or if the correct version of Node.js is present. Everything is baked into the container.

---

## 2. The Dockerfile

To create a Docker container, we write a recipe called a `Dockerfile`. This recipe tells Docker exactly how to construct the operating system from scratch.

Create a file named `Dockerfile` in the root of your project:

```dockerfile
# 1. Start from an official Node.js image based on Debian Linux
FROM node:18-bullseye
# 2. Install Google Chrome explicitly inside the Linux container
RUN apt-get update && apt-get install -y wget gnupg \
    && wget -q -O - https://dl-ssl.google.com/linux/linux_signing_key.pub | apt-key add - \
    && sh -c 'echo "deb [arch=amd64] http://dl.google.com/linux/chrome/deb/ stable main" >> /etc/apt/sources.list.d/google.com.list' \
    && apt-get update \
    && apt-get install -y google-chrome-stable
# 3. Create a working directory for our code
WORKDIR /usr/src/app
# 4. Copy the package.json and install NPM dependencies
COPY package*.json ./
RUN npm ci
# 5. Copy the rest of our TypeScript/Cucumber framework into the container
COPY . .
# 6. Set the default command to run when the container starts
CMD ["npm", "run", "test:bdd"]
```

---

## 3. Building and Running the Image

Now that we have our recipe, we tell Docker to "build" the image:

```bash
docker build -t my-selenium-framework .
```
*(This will take a few minutes as Docker literally downloads a Linux OS, installs Chrome, and installs your node_modules!)*

Once the image is built, you can execute it on *any* computer in the world that has Docker installed:

```bash
docker run -e ADMIN_PASSWORD="mySecretPassword" my-selenium-framework
```
Notice how we can securely pass environment variables (`-e`) directly into the container at runtime!

---

## 4. Docker in GitHub Actions

Because our entire environment (Node, Chrome, libraries) is bundled inside the container, our GitHub Actions workflow becomes incredibly simple. We don't need `setup-node` or `apt-get install chrome` anymore!

```yaml
name: Dockerized CI Pipeline
on:
  push:
    branches: [ "main" ]
jobs:
  docker-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Build Docker Image
        run: docker build -t my-selenium-framework .
      - name: Execute Tests inside Container
        env:
          ADMIN_PASSWORD: ${{ secrets.ADMIN_PASSWORD }}
        run: docker run -e ADMIN_PASSWORD my-selenium-framework
```

## Conclusion

Docker is the final evolution of modern test automation architecture. 

By containerizing your Selenium framework, you achieve 100% execution consistency. If a test passes on your laptop, you can mathematically guarantee it will pass in the CI pipeline, because they are running inside the exact same virtual operating system.

We have reached the end of our GitHub Actions journey! But what if your company doesn't use GitHub Actions? What if they use the oldest, most battle-tested CI server on the planet?

In our final tutorial of Phase 11, we will provide a brief **Overview of Jenkins Pipelines** to ensure you are ready for any enterprise environment!
