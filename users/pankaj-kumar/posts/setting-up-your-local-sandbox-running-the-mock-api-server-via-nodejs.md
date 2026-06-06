---
title: Setting Up Your Local Sandbox: Running the Mock API Server via Node.js
date: 14-Jan-2025
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [api-testing, automation, nodejs, mock-server]
category: API Testing
categories: [API Testing, Automation]
excerpt: >-
  When you're diving into API testing—whether you're using RestAssured, Supertest, Postman, or Karate—the absolute best way to learn is by having your own local sandbox.
readTime: 3 min read
---

When you're diving into API testing—whether you're using RestAssured, Supertest, Postman, or Karate—the absolute best way to learn is by having your own **local sandbox**. 

Relying on public, free APIs from the internet can be incredibly frustrating. They often go down, enforce strict rate limits (blocking you after 50 requests), or silently change their data, which instantly breaks your perfectly written test scripts.

To solve this, we have built the open-source **MyCodeYatra Mock API Server**. It is a fully featured backend running locally on your machine that simulates a real-world production environment. 

In this tutorial, we will walk through exactly how to download, install, and run this server using Node.js. Even if you have never used Node.js before, this guide will take you step-by-step!

> **Important Resource:** The entire codebase for this tutorial is hosted on our official GitHub Repository. You can find it here: [https://github.com/MYCodeYatra/myct-api-test-server](https://github.com/MYCodeYatra/myct-api-test-server)

---

## Step 1: Install Node.js
Because our Mock Server is built using Express.js (a popular Node.js framework), your computer needs to have Node.js installed to run it.

First, go to the official [Node.js website](https://nodejs.org/) and download the **LTS (Long Term Support)** version. This is the most stable version for Windows, Mac, or Linux. 

Run the downloaded installer and keep clicking "Next" to accept all the default settings.

To verify the installation, open your computer's Terminal (or Command Prompt) and type the following command:

```bash
node -v
```

If it prints a version number (like `v22.14.0`), you are good to go!

---

## Step 2: Download the Mock Server
You need to pull the code from our official GitHub organization to your local machine. You can do this via Git, or by directly downloading the ZIP file.

**Using Git (Recommended)**

If you have Git installed, simply open your terminal and run the following repository cloning command:

```bash
git clone https://github.com/MYCodeYatra/myct-api-test-server.git
```

**Downloading the ZIP File**

If you don't have Git, navigate to the GitHub repository at `https://github.com/MYCodeYatra/myct-api-test-server` in your browser. Click the green **"<> Code"** button, select **"Download ZIP"**, and extract the folder on your computer.

---

## Step 3: Install the Server Dependencies
Now that you have the code, you need to tell Node.js to download all the required third-party libraries (like `express` for the server, and `faker` for generating fake database data).

Open your terminal and navigate into the downloaded folder using the command below:

```bash
cd path/to/myct-api-test-server
```

Once inside the folder, run the Node Package Manager installation command:

```bash
npm install
```

*Note: `npm` automatically reads the `package.json` file inside the folder and downloads everything the server needs to run.*

---

## Step 4: Start the Server!
Once the installation finishes, starting the server is incredibly simple.

In the exact same terminal window, run this execution command:

```bash
node index.js
```

You should instantly see the following success messages in your terminal confirming the ports are active:

```text
🚀 REST Mock API Server running at http://localhost:8080
🚀 GraphQL Endpoint ready at http://localhost:8080/graphql
🚀 WebSocket Server listening at ws://localhost:8080/ws/chat
```

Congratulations! You now have a production-grade backend running right on your laptop!

---

## Step 5: Verify the API is Working
Before we start writing automation scripts in future blogs, let's manually verify that our API is successfully accepting requests.

Open your web browser (Chrome, Firefox, or Safari) and type the following URL into the address bar:
`http://localhost:8080/api/users`

Press **Enter**. You should instantly see a large JSON object returned to your screen containing realistic, dynamically generated user data! It will look something like this:

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

If you see this data, your server is working perfectly! Our server automatically generates 50 unique users every single time it boots up.

### Wrapping Up
You now have a reliable, isolated sandbox environment. Because the server runs locally, your tests will execute in milliseconds without any network latency, and you'll never hit an API rate limit again!

In the **next blog** of this series, we will learn how to bypass Node.js entirely and run this server in a fully containerized **Docker** environment!
