---
title: Mastering Postman: Data-Driven Testing in Newman
date: 2025-01-10
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [postman, api-testing, automation, newman, data-driven]
category: API Testing
categories: [Api Testing, Postman, Automation]
excerpt: >-
  Mastering Postman: Data-Driven Testing in Newman  > Key insight: You shouldn't duplicate a test request 100 times just to test 100 different input values. You write the test once, and run it 100 times
readTime: 2 min read
---
> **Important Update:** To get the most out of this tutorial, we highly recommend running the official [MyCodeYatra Mock API Server](https://github.com/MYCodeYatra/myct-api-test-server) locally on `http://localhost:8080`. Replace any generic public API URLs in these examples with your local Mock Server endpoints!



# Mastering Postman: Data-Driven Testing in Newman

> **Key insight:** You shouldn't duplicate a test request 100 times just to test 100 different input values. You write the test once, and run it 100 times using a data file.

Testing an API endpoint with one set of credentials proves it works. Testing it with 50 different edge-case inputs proves it is reliable.

If you have an endpoint like `POST /login`, you probably want to test:
1. Valid email, valid password (200 OK)
2. Invalid email, valid password (400 Bad Request)
3. Valid email, wrong password (401 Unauthorized)
4. Empty payload (400 Bad Request)

Instead of creating four separate requests in Postman, we can use **Data-Driven Testing** to inject an external CSV or JSON file into a single request.

## 1. Preparing the Data File

Let's create a simple CSV file named `login_test_data.csv`. The first row contains our variable names, and every subsequent row represents one "iteration" of the test.

```csv
email,password,expected_status
admin@test.com,SuperSecret123,200
bad_user@test.com,SuperSecret123,400
admin@test.com,WrongPass,401
,,400
```

## 2. Using Data Variables in Postman

To use this data in your Postman Collection, you treat the CSV column headers exactly like Environment variables by wrapping them in double curly braces.
In your request body, write:

```json
{
    "email": "{{email}}",
    "password": "{{password}}"
}
```

Then, in your **Tests** tab, you can access the `expected_status` column using `pm.iterationData.get()` to assert that the API returns the correct status code for that specific row:

```javascript
// Get the expected status code from the current CSV row
const expectedStatus = pm.iterationData.get("expected_status");
// Assert the API returned that status code
pm.test(`Status should be ${expectedStatus}`, function () {
    pm.response.to.have.status(parseInt(expectedStatus));
});
```

## 3. Running Data Files with Newman

To execute this data-driven test, we use the `newman run` command we learned in the last tutorial, but we append the `-d` (data) flag followed by our CSV file.
When Newman sees this flag, it will loop through the entire Collection **once for every row** in the CSV file.

```bash
newman run login_collection.json -e dev_env.json -d login_test_data.csv
```

As the terminal output scrolls by, you will see Newman execute the `POST /login` request four separate times. It will inject the data from row 1, run the tests, print the results, and then immediately loop back to start over with row 2.

## Final Takeaways

Data-driven testing allows you to separate your test logic from your test data. It keeps your Postman Collections incredibly lightweight while allowing QA engineers to add hundreds of new test cases simply by adding rows to an Excel spreadsheet. Now that we can run massive data sets from the command line, it's time to generate some beautiful HTML reports in our next post!
