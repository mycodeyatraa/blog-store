---
title: Mastering Postman: Pre-request Scripts for Dynamic Data
date: 2025-01-03
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [postman, api-testing, automation, javascript]
category: API Testing
categories: [Api Testing, Postman, Automation]
excerpt: >-
  Mastering Postman: Pre-request Scripts for Dynamic Data  > Key insight: Stop hardcoding email addresses and timestamps in your JSON body. By mastering Pre-request Scripts, your API collections become 
readTime: 2 min read
---

# Mastering Postman: Pre-request Scripts for Dynamic Data

> **Key insight:** Stop hardcoding email addresses and timestamps in your JSON body. By mastering Pre-request Scripts, your API collections become truly autonomous and reusable.

Have you ever hit an API endpoint that requires a uniquely generated `email` or an ever-changing `timestamp` for every single request? If you manually edit the JSON body every time you hit "Send," you are not testing—you are doing data entry. 

Postman offers a powerful feature known as **Pre-request Scripts**. Written in pure JavaScript, these scripts run *before* your API request is fired over the network, allowing you to manipulate variables and generate dynamic payloads on the fly.

## 1. The Execution Flow

To understand Pre-request scripts, you must understand the Postman execution order.


![diagram_1](https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/mastering-postman-pre-request-scripts-for-dynamic-data/images/diagram_1.png)


1. **Pre-request Script:** Generates data, fetches tokens, or calculates signatures.
2. **Network Request:** Injects the variables from Step 1 into the URL, Headers, or Body.
3. **Tests Script:** Validates the response from the server.

## 2. Generating Dynamic Data

Let's look at the most common use case: creating a new user where the `email` must be unique. 

Instead of typing a new email every time, we can use Postman's built-in `pm` object to generate a random string and store it as a local variable.

### In your Pre-request Script tab:

```javascript
// Generate a random 6-character string
const randomString = Math.random().toString(36).substring(2, 8);
const dynamicEmail = `testuser_${randomString}@mycodeyatra.com`;

// Store it in a variable scoped to this specific execution
pm.variables.set("dynamic_email", dynamicEmail);

// Generate a current ISO timestamp
const currentTimestamp = new Date().toISOString();
pm.variables.set("current_timestamp", currentTimestamp);
```

### In your Body (raw JSON) tab:

Now, simply reference those variables using the standard `{{variable_name}}` syntax:

```json
{
    "name": "John Doe",
    "email": "{{dynamic_email}}",
    "created_at": "{{current_timestamp}}"
}
```

Every time you click "Send," a fresh email and exact timestamp are generated instantaneously.

## 3. Postman's Built-in Dynamic Variables

If you don't want to write custom JavaScript, Postman actually includes the Faker.js library under the hood. You can use these directly in your JSON body without *any* pre-request scripts!

*   `{{$guid}}`: A v4 style GUID
*   `{{$timestamp}}`: Current Unix timestamp in seconds
*   `{{$randomEmail}}`: A random email address
*   `{{$randomFirstName}}`: A random first name

**Example Payload:**
```json
{
    "id": "{{$guid}}",
    "first_name": "{{$randomFirstName}}"
}
```

## Final Takeaways

Pre-request scripts unlock the true power of automation in Postman. By shifting data generation to code rather than manual input, you ensure your collections can run flawlessly in a CI/CD pipeline without human intervention. In our next topic, we will explore how to write assertions against the server's response!
