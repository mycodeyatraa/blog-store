---
title: Mocking Axios / Fetch Requests
date: 03-Mar-2025
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [supertest, jest, mocking, axios, testing]
category: API Testing
categories: [API Testing, NodeJS, Best Practices, Advanced]
excerpt: >-
  Keep your API tests fast, deterministic, and free from third-party rate limits. Learn how to intercept outbound network requests using jest.mock().
readTime: 5 min read
---

In modern microservice architectures, your API doesn't live in isolation. Your Express server likely makes outbound HTTP requests to external databases, payment gateways (Stripe), or third-party APIs (Weather, GitHub).

When running SuperTest, you **do not** want your test suite hitting a live Stripe API and charging real credit cards! We must intercept these outbound requests using `jest.mock()`.

---

## 1. Setting up the Dummy Gateway

Let's assume our Express API has an endpoint `/api/weather` that internally calls `https://api.weather.com` using the `axios` library.

We want to test our Express route, but we want to completely fake the response from `api.weather.com`.

## 2. Using jest.mock()

At the very top of your test file, you instruct Jest to hijack the module system. Whenever your Express app tries to `require('axios')`, Jest will secretly swap it out with a mock object.

```typescript
import request from 'supertest';
import express from 'express';
import axios from 'axios';
// 1. Tell Jest to intercept all calls to the 'axios' module
jest.mock('axios');
// Typecast it so TypeScript knows it has mock methods like .mockResolvedValueOnce
const mockedAxios = axios as jest.Mocked<typeof axios>;
// 2. Build our Express App
const app = express();
app.get('/api/weather', async (req, res) => {
    try {
        // In reality, this would hit the live third-party service
        const response = await axios.get('https://api.weather.com/v1/current');
        res.json({ source: 'WeatherAPI', data: response.data });
    } catch (error) {
        res.status(500).json({ error: 'Failed to fetch weather' });
    }
});
```

## 3. Mocking Success and Failure

Now, inside our `it()` blocks, we can dictate exactly what `axios.get` should return when our Express app calls it!

```typescript
describe('Mocking Axios / Fetch Requests', () => {
    beforeEach(() => {
        // Clear mock counters before every test to ensure isolation
        jest.clearAllMocks();
    });
    it('1. Mock a successful Axios GET request', async () => {
        // Provide the fake response data that Axios will return when called
        const fakeWeatherResponse = { data: { temperature: 72, condition: 'Sunny' } };
        mockedAxios.get.mockResolvedValueOnce(fakeWeatherResponse);
        // Run SuperTest against our app (which triggers the mocked Axios call)
        const response = await request(app).get('/api/weather');
        expect(response.status).toBe(200);
        expect(response.body.data.temperature).toBe(72);
        // Assert that the app actually attempted to call the correct external URL
        expect(mockedAxios.get).toHaveBeenCalledWith('https://api.weather.com/v1/current');
        expect(mockedAxios.get).toHaveBeenCalledTimes(1);
    });
    it('2. Mock an Axios network failure (500 Server Error)', async () => {
        // Force Axios to throw a network error to simulate the third-party API being down
        mockedAxios.get.mockRejectedValueOnce(new Error('Network Timeout'));
        const response = await request(app).get('/api/weather');
        // Assert our Express app handled the failure gracefully
        expect(response.status).toBe(500);
        expect(response.body.error).toBe('Failed to fetch weather');
    });
});
```

## 4. Execution Results

When we execute the test, our Express route successfully handles the mocked data, and we achieve 100% test coverage without making a single real network call!

```bash
PASS tests/mocking.test.ts
  Mocking Axios / Fetch Requests
    [PASS] 1. Mock a successful Axios GET request (95 ms)
    [PASS] 2. Mock an Axios network failure (500 Server Error) (16 ms)
Test Suites: 1 passed, 1 total
Tests:       2 passed, 2 total
Snapshots:   0 total
Time:        4.918 s
```

## 5. Conclusion

Mocking dependencies is crucial for keeping your test suites fast, deterministic, and isolated from external network failures.

In our next tutorial, we will learn how to measure the effectiveness of our tests using **Generating Test Coverage Reports**!
