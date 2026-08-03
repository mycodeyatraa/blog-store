---
title: Advanced Authentication: OAuth 2.0 & SSO in Karate
date: 26-Feb-2026
lastUpdated: 26-Feb-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: ["karate", "oauth2", "sso", "authentication", "jwt"]
category: API Karate
categories: ["API Karate", "API Testing", "Security"]
excerpt: >-
  Automate secure API authentication! Learn how to handle OAuth 2.0 token exchanges, JWT headers, and SSO flows in Karate.
readTime: 8 min read
---

# Advanced Authentication: OAuth 2.0 & SSO in Karate

Enterprise REST APIs require secure authorization tokens (OAuth 2.0, OpenID Connect, JWT) for every request.

In **Karate DSL**, you can create modular authentication feature files that automatically acquire access tokens and inject `Authorization: Bearer <token>` headers into downstream test scenarios.

---

## 1. Modular Token Acquisition Feature

Create a dedicated reusable token generation feature:

**features/auth/get-token.feature**

```gherkin
@ignore
Feature: OAuth 2.0 Access Token Generator
 
  Scenario: Acquire Bearer Token
    Given url 'https://mycodeyatra.com/oauth/token'
    And form field grant_type = 'client_credentials'
    And form field client_id = karate.properties['client.id'] || 'automation_client'
    And form field client_secret = karate.properties['client.secret'] || 'secret_123'
    When method post
    Then status 200
    * def token = response.access_token
```

---

## 2. Reusing Authentication Across Feature Files

Invoke the token feature using `karate.callSingle()` in `karate-config.js`:

**karate-config.js**

```javascript
function fn() {
  var config = {
    baseUrl: 'https://mycodeyatra.com/api'
  };
 
  // Call authentication feature once globally
  var authResult = karate.callSingle('classpath:features/auth/get-token.feature', config);
  config.authToken = authResult.token;
 
  // Set global authorization header
  karate.configure('headers', { Authorization: 'Bearer ' + config.authToken });
 
  return config;
}
```

---

## 3. Executing Protected API Tests

Now every feature scenario automatically includes the valid Bearer token without repetitive setup code:

**features/secure-data.feature**

```gherkin
Feature: Protected Resource Validation
 
  Scenario: Fetch User Confidential Details
    Given url baseUrl + '/v1/user/profile'
    When method get
    Then status 200
    And match response.email == '#present'
    And match response.roles contains 'ADMIN'
```
