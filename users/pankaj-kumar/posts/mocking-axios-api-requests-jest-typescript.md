---
title: API Mocking: Testing Edge Cases without a Real Server
date: 11-Dec-2024
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [typescript, selenium, api-testing, jest, mocking, axios]
category: Selenium TypeScript
categories: [Selenium TypeScript, API Testing Integration]
excerpt: >-
  Learn to mock backend API responses using Jest and Axios to test 500 errors and edge cases without needing a real broken server.
readTime: 4 min read
---

# API Mocking: Testing Edge Cases without a Real Server

One of the biggest challenges in API testing is testing **Edge Cases**. 

How do you verify that your automation framework correctly handles a `500 Internal Server Error`? You can't just ask the backend developers to intentionally break the staging server for you!

The solution is **API Mocking**. By intercepting the network requests and returning fake responses, we can simulate any scenario we want—instantly and reliably. 

In this tutorial, we will use Jest's built-in mocking capabilities to mock our Axios HTTP requests.

---

## 1. Why Mock APIs?

1. **Simulate Errors:** Easily test how your framework handles `500 Server Errors`, `429 Too Many Requests` (Rate Limiting), or `401 Unauthorized` without actually triggering them on a live system.
2. **Frontend Independence:** Test your UI or integration layers even when the backend server is down or under development.
3. **Speed:** Mocked requests resolve in milliseconds because they never actually cross the network.

---

## 2. Writing the Mocking Tests

Jest makes mocking third-party libraries like `axios` incredibly simple. You do not need to install heavy external tools like `wiremock` for basic unit/integration mocking.

Create `tests/api_mocking.test.ts`:

```typescript
import axios from "axios";
import { ApiUtils } from "./utils/ApiUtils";
// 1. Tell Jest to mock the entire axios library
jest.mock("axios");
const mockedAxios = axios as jest.Mocked<typeof axios>;
describe("Phase 6 - API Mocking with Jest", () => {
  afterEach(() => {
    // Reset mocks after each test to prevent pollution
    jest.clearAllMocks();
  });
  it("Should mock a successful GET request (200 OK)", async () => {
    // 2. Define the fake data we want axios to return
    const mockResponse = {
      data: { id: 999, title: "Mocked Title", body: "Mocked Body" },
      status: 200,
      statusText: "OK",
      headers: {},
      config: {}
    };
    // 3. Instruct the mocked axios to resolve with our fake data
    mockedAxios.get.mockResolvedValueOnce(mockResponse);
    // 4. Call our ApiUtils (which calls the mocked axios internally)
    const response = await ApiUtils.get("/posts/999");
    // 5. Validate the response matches our mock!
    expect(response.status).toBe(200);
    expect(response.data.title).toBe("Mocked Title");
    // 6. Verify that axios.get was actually called with the correct URL
    expect(mockedAxios.get).toHaveBeenCalledWith(expect.stringContaining("/posts/999"), expect.any(Object));
    console.log(`Successfully mocked GET response for ID: ${response.data.id}`);
  });
  it("Should mock a 500 Internal Server Error to test error handling", async () => {
    // 2. Define a fake error response
    const mockErrorResponse = {
      response: {
        data: { error: "Database Connection Failed" },
        status: 500
      },
      message: "Request failed with status code 500"
    };
    // 3. Instruct mocked axios to reject with the fake error
    mockedAxios.get.mockRejectedValueOnce(mockErrorResponse);
    // 4. Call our ApiUtils (it should catch the error and return the response object)
    const response = await ApiUtils.get("/posts/error");
    // 5. Validate our framework handles the 500 error gracefully
    expect(response.status).toBe(500);
    expect(response.data.error).toBe("Database Connection Failed");
    console.log(`Successfully mocked and handled a 500 Internal Server Error`);
  });
});
```

### Breaking it down:
* `jest.mock("axios")`: This hijacks the actual Axios package. Any file that imports `axios` during this test run will receive a fake Jest function instead of the real library.
* `mockResolvedValueOnce()`: Tells the fake Axios to instantly return a success promise with our predefined data on the very next call.
* `mockRejectedValueOnce()`: Tells the fake Axios to throw an error, simulating a server crash or network failure.

---

## 3. Test Execution Output

Run the test:

```bash
> ENV_NAME=qa jest tests/api_mocking.test.ts
```

Output:

```text
 PASS  tests/api_mocking.test.ts
  Phase 6 - API Mocking with Jest
    √ Should mock a successful GET request (200 OK) (15 ms)
    √ Should mock a 500 Internal Server Error to test error handling (5 ms)
  console.log
    [API] Sending GET request to: https://jsonplaceholder.typicode.com/posts/999
    [API] Received Status: 200
    Successfully mocked GET response for ID: 999
  console.log
    [API] Sending GET request to: https://jsonplaceholder.typicode.com/posts/error
    [API] GET request failed: Request failed with status code 500
    Successfully mocked and handled a 500 Internal Server Error
```

Notice the execution times: **15 ms** and **5 ms**. Because no actual network request was made, the tests execute at the speed of your CPU!

## Conclusion

Mocking is an essential tool in a Senior SDET's toolkit. It allows you to test your framework's resiliency against unpredictable backend behaviors without relying on volatile test environments.

In our next lesson, we will move on to **Contract Testing**, where we ensure the API response structure conforms strictly to a defined schema!
