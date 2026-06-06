---
title: Testing WebSocket Connections
date: 26-Feb-2025
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [supertest, websockets, ws, real-time, jest]
category: API Testing
categories: [API Testing, NodeJS, WebSockets, Automation]
excerpt: >-
  Learn how to overcome SuperTest's HTTP limitations by integrating the 'ws' library to automate real-time WebSocket communication in Jest.
readTime: 5 min read
---

SuperTest is fundamentally built around the HTTP `request/response` lifecycle. However, modern applications increasingly rely on **WebSockets (WS)** for real-time, bidirectional communication (e.g. Chat Applications, Live Trading Tickers, Multi-player Gaming).

Because WebSockets hold persistent connections open rather than closing them immediately after a response, SuperTest cannot test them directly. 

In this tutorial, we will learn how to seamlessly integrate the popular `ws` NPM library into our Jest suite to run WebSocket tests alongside our SuperTest API validations.

---

## 1. Setup and Installation

First, we need to install the WebSocket client library and its TypeScript definitions:

```bash
npm install -D ws @types/ws
```

## 2. Writing the WebSocket Tests

When testing WebSockets, it is critical to properly handle connection setup and teardown. Otherwise, Jest will hang indefinitely because the persistent socket connection keeps the Node process alive!

We use Jest's `beforeEach` and `afterEach` hooks along with the `done` callback to safely manage the socket lifecycle.

```typescript
import WebSocket from 'ws';
describe('Testing WebSocket Connections', () => {
    const WS_URL = 'ws://localhost:8080/ws/chat';
    let ws: WebSocket;
    // Open connection before each test
    beforeEach((done) => {
        ws = new WebSocket(WS_URL);
        ws.on('open', done);
    });
    // Cleanly close connection after each test
    afterEach((done) => {
        if (ws.readyState === WebSocket.OPEN) {
            ws.close();
        }
        done();
    });
    it('1. Connect to WebSocket and receive welcome message', (done) => {
        // Listen for the first message emitted by the server upon connection
        ws.on('message', (data) => {
            const parsed = JSON.parse(data.toString());
            expect(parsed.type).toBe('welcome');
            expect(parsed.message).toBe('Connected to Mock Chat Server!');
            done(); // Tell Jest the test is complete!
        });
    });
    it('2. Send Message and receive Echo', (done) => {
        let messageCount = 0;
        ws.on('message', (data) => {
            messageCount++;
            // We ignore the first 'welcome' message and assert on the second
            if (messageCount === 2) {
                const parsed = JSON.parse(data.toString());
                expect(parsed.type).toBe('echo');
                expect(parsed).toHaveProperty('timestamp');
                expect(JSON.parse(parsed.data).action).toBe('ping');
                done(); // Tell Jest the test is complete!
            }
        });
        // Push data to the server
        ws.send(JSON.stringify({ action: 'ping' }));
    });
});
```

## 3. The Power of the `done()` Callback

If you look closely at the code above, we pass a `done` parameter to the `it()` block. 

WebSockets are event-driven. Jest has no way of knowing when our assertions inside `ws.on('message', ...)` are actually finished. If we don't explicitly call `done()`, Jest will either exit early or timeout! By calling `done()`, we manually signal that the asynchronous event has been validated.

## 4. Execution Results

Let's execute `npx jest tests/websocket.test.ts` to see our WebSocket hooks in action:

```bash
PASS tests/websocket.test.ts
  Testing WebSocket Connections
    [PASS] 1. Connect to WebSocket and receive welcome message (76 ms)
    [PASS] 2. Send Message and receive Echo (12 ms)
Test Suites: 1 passed, 1 total
Tests:       2 passed, 2 total
Snapshots:   0 total
Time:        4.109 s
```

## 5. Conclusion

By combining the fluent HTTP assertions of SuperTest with the robust event-driven nature of the `ws` library, your Jest suite is now fully equipped to test the entirety of a modern backend architecture.

In our next lesson, we will pivot to exploring **File Uploads and Downloads**!
