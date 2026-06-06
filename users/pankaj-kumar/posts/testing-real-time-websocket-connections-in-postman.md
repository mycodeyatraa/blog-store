---
title: Testing Real-Time WebSocket Connections in Postman
date: 2025-01-28
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [postman, websockets, automation]
category: API Testing
categories: [API Testing, Automation, Postman]
excerpt: >-
  Postman isn't just for HTTP requests. It has robust, native support for WebSockets. In this tutorial, we will use Postman to connect to a live WebSocket server.
readTime: 5 min read
---

In traditional REST APIs, the client sends a request and waits for a response. But modern applications often require real-time, two-way communication. Think of chat applications, live stock tickers, or multiplayer games. These rely on **WebSockets**.

In this tutorial, we will use Postman to connect to a live WebSocket server using the MyCodeYatra Mock API Server we set up previously.

---

## Establishing a WebSocket Connection in Postman

Postman isn't just for HTTP requests. It has robust, native support for WebSockets.

First, ensure your MyCodeYatra Mock API Server is running locally on port `8080`. 

Next, open Postman and click on the **New** button. Instead of selecting an HTTP request, select **WebSocket Request**.

In the URL bar, you cannot use `http://`. WebSockets use their own protocol. Enter the following URL to connect to our mock server's chat room:

`ws://localhost:8080/ws/chat`

Click the **Connect** button.

Immediately, Postman opens a persistent tunnel to the server. You will see a "Connected" status, and the server will instantly push a welcome message to your Messages panel!

---

## Sending and Receiving Real-Time Messages

Now that the tunnel is open, you can push data to the server at any time without establishing a new connection.

Our local mock server acts as an "Echo Server". Any message you send will be immediately broadcast back to you.

Navigate to the **Message** tab just below the URL bar. Type the following text:

`Hello Server! Are you there?`

Click the **Send** button. 

Watch the Messages panel. You will see your sent message appear with an upward arrow. Instantly, a downward arrow will appear, containing the server's echoed response!

---

## Why Test WebSockets in Postman?

Testing WebSockets manually is crucial before you attempt to automate them. Because WebSockets don't have standard HTTP status codes or immediate responses, you need to verify the connection stability, payload formats, and server behavior before writing automated assertions.

By saving this WebSocket request in your Postman Collection, you ensure that your team has a standardized, repeatable way to verify real-time backend infrastructure.

In the **next blog**, we will explore another advanced protocol supported by Postman: **GraphQL**!
