---
title: Mastering Postman: OAuth 2.0 & Silent Token Refresh
date: 2025-01-07
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [postman, api-testing, automation, oauth, security]
category: API Testing
categories: [Api Testing, Postman, Automation]
excerpt: >-
  Mastering Postman: OAuth 2.0 & Silent Token Refresh  > Key insight: If you are manually clicking "Get New Access Token" every time your OAuth token expires, your tests are not truly automated. Silent 
readTime: 3 min read
---
> **Important Update:** To get the most out of this tutorial, we highly recommend running the official [MyCodeYatra Mock API Server](https://github.com/MYCodeYatra/myct-api-test-server) locally on `http://localhost:8080`. Replace any generic public API URLs in these examples with your local Mock Server endpoints!



# Mastering Postman: OAuth 2.0 & Silent Token Refresh

> **Key insight:** If you are manually clicking "Get New Access Token" every time your OAuth token expires, your tests are not truly automated. Silent refresh is mandatory for CI/CD.

We are officially entering Phase 2 of our Postman Mastery series! We've covered variables, assertions, and basic request chaining. Now we are going to tackle one of the most frustrating aspects of API testing: **OAuth 2.0**.

OAuth tokens usually expire after an hour. If you run your collection locally, a failed token is a minor annoyance. If you run your collection in a midnight Jenkins pipeline and it fails because the token expired, you have a broken build.

## 1. The Standard OAuth Flow (Manual)

Postman has a built-in Authorization tab specifically for OAuth 2.0. You configure your Auth URL, Access Token URL, Client ID, and Client Secret. 

You click **Get New Access Token**, a browser window pops up, you log in, and Postman saves the token.

**The Problem:** This requires human interaction. A headless CI/CD server cannot click a popup window.

## 2. Automating the Client Credentials Flow

If your API supports the `client_credentials` grant type (usually used for server-to-server communication), you can completely bypass the popup by requesting the token programmatically.

Instead of using Postman's Auth tab, we are going to write a **Pre-request Script** on the Collection level. This ensures that *every* request checks for a valid token before firing.


![diagram_1](https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/mastering-postman-oauth-20-silent-token-refresh/images/diagram_1.png)


### The Collection Pre-request Script

Here is the exact code you place in your Collection's Pre-request tab:

```javascript
// 1. Check if we already have a valid token
const tokenDate = pm.environment.get("token_expiry_date");
const currentToken = pm.environment.get("access_token");
//
// Calculate expiration using subtraction to ensure compatibility
const timeRemaining = new Date(tokenDate) - new Date();
//
if (!currentToken || !tokenDate || timeRemaining <= 0) {
    console.log("Token expired or missing. Fetching new token...");
    
    // 2. Define the Auth Request
    const authRequest = {
        url: 'https://auth.mycodeyatra.com/oauth/token',
        method: 'POST',
        header: 'Content-Type: application/x-www-form-urlencoded',
        body: {
            mode: 'urlencoded',
            urlencoded: [
                {key: "grant_type", value: "client_credentials"},
                {key: "client_id", value: pm.environment.get("client_id")},
                {key: "client_secret", value: pm.environment.get("client_secret")}
            ]
        }
    };
//
    // 3. Send the Auth Request asynchronously
    pm.sendRequest(authRequest, function (err, res) {
        if (err) {
            console.error("Failed to fetch token", err);
        } else {
            const responseJson = res.json();
            
            // 4. Save the new token
            pm.environment.set("access_token", responseJson.access_token);
            
            // 5. Calculate expiry time (subtracting 1 min for safety buffer)
            const expiryDate = new Date();
            expiryDate.setSeconds(expiryDate.getSeconds() + responseJson.expires_in - 60);
            pm.environment.set("token_expiry_date", expiryDate.toISOString());
            
            console.log("New token successfully acquired and saved.");
        }
    });
} else {
    console.log("Current token is still valid. Proceeding with request.");
}
```

## 3. Injecting the Automated Token

Now that our Collection Pre-request script guarantees an `access_token` is always available in the environment, you simply go to your Collection's Authorization tab:

1. Type: **Bearer Token**
2. Token: `{{access_token}}`

Every single request in your collection will now inherit this token. When you hit "Send", the Pre-request script checks the timestamp. If it's expired, it pauses your request, fetches a new token in the background, updates the variable, and *then* fires your actual request.

## Final Takeaways

By moving OAuth token generation out of the UI and into a Pre-request script, your Postman collections become entirely self-sufficient. This is the absolute prerequisite for running tests via the command line—which is exactly what we will cover in our upcoming tutorial on Newman!
