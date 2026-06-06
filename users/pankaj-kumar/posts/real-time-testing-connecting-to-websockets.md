---
title: Real-Time Testing: Connecting to WebSockets
date: 2025-01-24
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [websockets, api-testing, automation, real-time, postman]
category: API Testing
categories: [API Testing, Automation, WebSockets]
excerpt: >-
  What if you are testing a live chat application or a stock ticker? To solve this, modern applications use WebSockets. In this tutorial, we will connect to the server's built-in WebSocket chat room.
readTime: 4 min read
---

Both REST and GraphQL share a common limitation: they rely on a request-response cycle. The client must explicitly ask the server for data before the server can reply. 

But what if you are testing a live chat application, a stock ticker, or a multiplayer game? The client cannot sit in a loop constantly asking the server if there are new messages. 

To solve this, modern applications use **WebSockets**. WebSockets establish a persistent, two-way connection between the client and the server, allowing data to flow freely in both directions in real-time. In this tutorial, we will connect to the MyCodeYatra Mock API Server's built-in WebSocket chat room!

> **Important Resource:** The entire codebase for this tutorial is hosted on our official GitHub Repository. You can find it here: [https://github.com/MYCodeYatra/myct-api-test-server](https://github.com/MYCodeYatra/myct-api-test-server)

---

## The WebSocket Protocol

Unlike standard HTTP endpoints that start with `http://` or `https://`, WebSocket endpoints use the `ws://` or `wss://` (secure) protocol.

When you start our mock server, you might notice this line in the startup logs: `WebSocket Server listening at ws://localhost:8080/ws/chat`. This is the exact endpoint we will use to test real-time communication.

Because WebSockets are persistent connections, we cannot use a simple tool like `curl` to test them. We need a specialized client.

---

## Connecting via Postman

Postman has excellent built-in support for WebSockets. We will use it to establish our first connection.

First, open Postman and click on **New**. Instead of selecting a standard HTTP request, select **WebSocket Request**. 

In the URL bar, type the server's WebSocket address: `ws://localhost:8080/ws/chat`.

Click the bright blue **Connect** button. 

Immediately, Postman will open a continuous tunnel to the server. You will see a "Connected" status indicator, and in the Messages panel at the bottom, the server will instantly push a welcome message to you:

```json
{
  "system": "Welcome to the Mock Chat! Send a message to see the echo."
}
```

---

## Sending and Receiving Real-Time Messages

Now that the connection is wide open, we can send data to the server at any time without establishing a new HTTP handshake.

Our mock server is programmed to act as an "Echo Server". Any message you send to it will be immediately broadcast back to you (and anyone else connected to the chat room).

In Postman, go to the **Message** tab (right below the URL bar). Type the following plain text message:

`Hello Server! Are you there?`

Click the **Send** button. 

Instantly, you will see two things happen in the Messages panel below. An upward arrow will appear showing the message you sent, and a downward arrow will appear showing the server instantly echoing your exact message back to you.

---

## Why This Matters for Automation

Testing WebSockets is notoriously difficult because standard assertions rely on immediate responses. With WebSockets, an automated test might send a message, but the server's response could arrive 2 seconds later, or not at all.

By practicing with our local echo server, you can configure your automation frameworks (like RestAssured, Cypress, or Playwright) to successfully open WebSocket streams, establish event listeners, and perform asynchronous assertions when the server pushes data back!

### Wrapping Up

WebSockets unlock the ability to test truly dynamic, real-time applications. You have now established a persistent connection, received unsolicited server data, and successfully transmitted bidirectional messages!

In the **next blog**, we will explore how to test asynchronous architectures. We will step away from WebSockets and dive into **Webhooks**, learning how servers notify external systems about background events!
