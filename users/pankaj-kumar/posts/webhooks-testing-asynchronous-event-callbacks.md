---
title: Webhooks: Testing Asynchronous Event Callbacks
date: 2025-01-25
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [webhooks, api-testing, automation, asynchronous, callbacks]
category: API Testing
categories: [API Testing, Automation, Webhooks]
excerpt: >-
  Keeping a WebSocket open 24/7 for a single, rare event is a massive waste of server resources. Instead, modern architectures use Webhooks to trigger asynchronous background events and listen for callbacks.
readTime: 4 min read
---

In the previous tutorial, we explored how WebSockets maintain an open, persistent connection for real-time data. But what if you only need the server to tell you when a specific event happens—like when a payment is processed or a video finishes rendering? 

Keeping a WebSocket open 24/7 for a single, rare event is a massive waste of server resources. Instead, modern architectures use **Webhooks**.

In this tutorial, we will use the MyCodeYatra Mock API Server to trigger an asynchronous background event and listen for a Webhook callback!

> **Important Resource:** The entire codebase for this tutorial is hosted on our official GitHub Repository. You can find it here: [https://github.com/MYCodeYatra/myct-api-test-server](https://github.com/MYCodeYatra/myct-api-test-server)

---

## What is a Webhook?

A Webhook is essentially a "reverse API". 

In a standard API, your client makes a request to the server, and the server answers. With a Webhook, you give the server a URL and say, "I am going to go do something else. When you are finished processing my data, send an HTTP POST request to this URL to let me know."

Testing Webhooks is challenging because you need a public URL that the server can reach to receive the callback.

---

## Setting Up a Webhook Listener

Since our mock server is running locally, we can easily set up a local listener without needing a public domain. 

The easiest way to catch a webhook locally is by using a free service like **Webhook.site** or by using Postman's built-in webhook listener. For this tutorial, we will use Webhook.site because it requires zero configuration.

Open your browser and navigate to `https://webhook.site`.

The website will instantly generate a unique, temporary URL for you (e.g., `https://webhook.site/YOUR-UNIQUE-ID`). Copy this URL and keep the browser tab open. This browser tab is now waiting to receive HTTP requests.

---

## Triggering the Asynchronous Event

Now we need to tell our Mock API Server to simulate a long-running background task, and we need to pass it our Webhook URL so it knows where to send the result when it finishes.

Our server provides a `/api/webhooks/trigger` endpoint. It expects a JSON payload containing a `targetUrl` and a `delay` (simulating the time the background task takes).

Open your terminal and run this command, making sure to replace the placeholder URL with your unique Webhook.site URL:

```bash
curl -X POST http://localhost:8080/api/webhooks/trigger \
-H "Content-Type: application/json" \
-d '{"targetUrl": "https://webhook.site/YOUR-UNIQUE-ID", "delay": 8000}'
```

The terminal will instantly return a `202 Accepted` response. It will not wait for the task to finish! The server has acknowledged the request and moved the work into a background queue.

```json
{
  "status": "processing",
  "message": "Task queued. A webhook will be sent to the target URL when complete."
}
```

---

## Catching the Callback

Now, switch back to the browser tab where you have Webhook.site open. 

Wait for exactly 8 seconds (the delay we requested). Suddenly, the screen will flash, and a brand new POST request will appear in the left-hand column!

Click on it to view the payload. You will see that the MyCodeYatra Mock API Server has successfully reached out across the internet and delivered the final payload:

```json
{
  "event": "task.completed",
  "data": {
    "taskId": "whk-1a2b3c4d",
    "status": "success",
    "completedAt": "2025-01-25T16:45:00.000Z"
  }
}
```

### Wrapping Up

You have successfully tested an asynchronous, event-driven architecture! 

By utilizing temporary webhook listeners, your automated test suites can trigger long-running asynchronous jobs on the backend, continue executing other tests, and then assert that the correct webhook payload was eventually delivered.

In the **next blog**, we will pull everything together. We will explore how to integrate our automated API tests directly into a **CI/CD Pipeline** so they run automatically on every code commit!
