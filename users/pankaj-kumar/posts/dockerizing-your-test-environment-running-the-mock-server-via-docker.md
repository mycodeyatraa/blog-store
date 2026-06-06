---
title: Dockerizing Your Test Environment: Running the Mock Server via Docker
date: 2025-01-15
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [docker, api-testing, automation, mock-server, containerization]
category: API Testing
categories: [API Testing, Automation, Docker]
excerpt: >-
  In the previous blog, we successfully set up our local mock API server using Node.js. While that works perfectly, configuring Node.js and managing dependencies can sometimes cause headaches.
readTime: 3 min read
---

In the previous blog, we successfully set up our local mock API server using Node.js. While that works perfectly, configuring Node.js, ensuring you have the right version, and managing `npm install` dependencies can sometimes cause headaches, especially when sharing the project with teammates who have different machine setups.

This is where **Docker** comes in. Docker allows you to package the entire server, its database, and all its dependencies into a single, isolated container. You don't even need Node.js installed on your computer to run it!

In this tutorial, we will walk through exactly how to containerize the MyCodeYatra Mock API Server so you can spin it up with a single command.

> **Important Resource:** The entire codebase for this tutorial is hosted on our official GitHub Repository. You can find it here: [https://github.com/MYCodeYatra/myct-api-test-server](https://github.com/MYCodeYatra/myct-api-test-server)

---

## Step 1: Install Docker Desktop

To run Docker containers, your machine needs the Docker Engine. 

Head over to the official [Docker website](https://www.docker.com/products/docker-desktop/) and download Docker Desktop for your operating system (Windows, Mac, or Linux).

Run the installer and follow the standard installation prompts. Once installed, launch the Docker Desktop application and ensure the engine is running (you should see a green icon in the bottom left corner of the app).

To verify the installation, open your Terminal or Command Prompt and type the following command:

```bash
docker -v
```

If it prints a version number like `Docker version 24.0.5`, you are ready for the next step!

---

## Step 2: Download the Mock Server Code

If you haven't already downloaded the server codebase from our previous tutorial, you need to pull it from our official GitHub organization.

Open your terminal and run the following command to clone the repository:

```bash
git clone https://github.com/MYCodeYatra/myct-api-test-server.git
```

Once the download is complete, navigate inside the project folder using this command:

```bash
cd myct-api-test-server
```

---

## Step 3: Build the Docker Image

Inside the repository, we have already provided a pre-configured `Dockerfile`. This file contains all the instructions Docker needs to build a custom snapshot (called an image) of our server.

From inside the `myct-api-test-server` folder, run the following build command:

```bash
docker build -t myct-api-test-server .
```

*Note: Do not forget the period `.` at the end of the command! That tells Docker to use the files in your current directory.*

This process will take a minute or two as Docker pulls a lightweight Linux image, installs Node.js inside it, and packages our server code.

---

## Step 4: Run the Docker Container

Now that we have built the image, it is time to bring it to life! We need to run the image inside an isolated container and tell Docker to expose port `8080` so our browser can access it.

Run the following command to start the container:

```bash
docker run -p 8080:8080 myct-api-test-server
```

You will instantly see the familiar startup logs in your terminal, confirming the server is actively listening from inside the container:

```text
🚀 REST Mock API Server running at http://localhost:8080
🚀 GraphQL Endpoint ready at http://localhost:8080/graphql
🚀 WebSocket Server listening at ws://localhost:8080/ws/chat
```

---

## Step 5: Verify the Containerized API

Even though the server is completely isolated inside a Docker container, it behaves exactly as if it were running natively on your machine!

Open your web browser and navigate to the following URL:
`http://localhost:8080/api/users`

Just like before, you should instantly see a large JSON object returned to your screen containing realistic, dynamically generated user data.

```json
{
  "page": 1,
  "limit": 10,
  "total": 50,
  "data": [
    {
      "id": "e4f8d2b2-6c8a-493b-b2f5-8d9e2b1a8c90",
      "name": "Alice Johnson",
      "email": "alice.j@example.com",
      "role": "admin",
      "createdAt": "2025-01-10T14:22:11.000Z"
    }
  ]
}
```

### Wrapping Up

You have successfully dockerized the MyCodeYatra Mock API Server! 

Because the server is now containerized, you can share this exact setup with your entire QA team. It guarantees that the server will run identically on every single machine, completely eliminating the classic "It works on my machine" problem.

In the **next blog** of this series, we will finally start hitting the server! We will explore the core REST endpoints and learn how to dynamically filter and search our fake user data.
