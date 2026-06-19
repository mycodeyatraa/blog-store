---
title: The Container Revolution: Docker for Selenium
date: 07-Feb-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [typescript, selenium, devops, docker, containers, automation, ci-cd]
category: Selenium TypeScript
categories: [Selenium TypeScript, DevOps]
excerpt: >-
  Kill the 'it works on my machine' excuse forever. Learn how to package your Node.js, TypeScript, and Cucumber automation framework into an immutable Docker container.
readTime: 4 min read
---

# The Container Revolution: Docker for Selenium

In the old days of software testing, setting up an automation environment took three days. You had to install Java, install Node.js, install the specific version of Chrome, download the exact matching ChromeDriver, configure the PATH variables, and pray that it worked.

And when it failed on a coworker's machine, the infamous excuse was born: *"It works on my machine."*

**Docker** fundamentally kills that excuse forever.

---

## 1. What is Docker?

Docker is a platform that packages an application and all its dependencies into a standardized unit called a **Container**. 

A container is essentially a lightweight, completely isolated mini-computer running inside your actual computer. Because the container contains its own Operating System, its own Node.js installation, and its own Chrome binaries, it is guaranteed to run identically on *every* machine on the planet—whether it's your laptop, your coworker's MacBook, or a massive Linux server in the cloud.

---

## 2. Dockerizing our TypeScript Executor

To use Docker, we must write a set of instructions telling Docker how to build our container. We do this by creating a `Dockerfile` in the root of our repository.

Here is the perfect `Dockerfile` for a Node.js/TypeScript automation framework:

```dockerfile
# 1. Start from an official Node.js image based on Alpine Linux (super lightweight)
FROM node:20-alpine
# 2. Set the working directory inside the container
WORKDIR /usr/src/app
# 3. Copy package.json and package-lock.json first (for caching optimization)
COPY package*.json ./
# 4. Install dependencies (including TypeScript, Cucumber, Selenium)
RUN npm ci
# 5. Copy the rest of our framework code (tests, page objects, etc.)
COPY . .
# 6. Define the command that runs when the container starts
CMD ["npm", "run", "test"]
```

### Understanding the Steps
Notice that we did *not* install Chrome or ChromeDriver in this `Dockerfile`. Why?

Because this container's *only* job is to be the **Executor**. It runs the Node.js process. It compiles the TypeScript. It parses the Cucumber feature files. 

But where do the tests actually run? They run on a Remote WebDriver (the Selenium Grid), which will exist in an entirely separate container. This architecture is called **Separation of Concerns**.

---

## 3. Building and Running the Image

Once you have your `Dockerfile`, you must "build" it into an Image.

Run this command in your terminal:

```bash
docker build -t my-selenium-framework:latest .
```
Docker will execute the steps in your `Dockerfile` one by one, downloading Node.js, running `npm ci`, and saving the final state as an immutable Image named `my-selenium-framework`.

To execute your tests, you run the container:

```bash
docker run --rm my-selenium-framework:latest
```
The `--rm` flag tells Docker to destroy the container immediately after the tests finish. We want a completely fresh, isolated environment every single time we run our tests!

---

## 4. The Power of Environment Variables

Because our container is immutable, we cannot change the code inside it without rebuilding it. So how do we tell the container to run on "QA" instead of "Staging", or point it to a different Selenium Grid URL?

We use Environment Variables at runtime:

```bash
docker run --rm \
  -e TEST_ENV=QA \
  -e GRID_URL=http://my-remote-grid:4444/wd/hub \
  my-selenium-framework:latest
```

Inside your TypeScript code, you simply read `process.env.TEST_ENV` and `process.env.GRID_URL` to dynamically alter your execution path!

## Conclusion

By wrapping your Node.js execution environment inside a Docker container, you have eliminated environment setup time and guaranteed that your tests run identically across the entire engineering department.

But how do we execute this container automatically every time a developer merges code? 

In our next tutorial, we will take our newly minted Docker image and integrate it into a cloud CI/CD pipeline: **GitHub Actions**.
