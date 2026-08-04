---
title: PUT/PATCH APIs - Playwright Java API Automation & Auth
date: 07-Feb-2026
lastUpdated: 07-Feb-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [playwright, java, junit5, api-testing, rest-api, authentication, mycodeyatra]
category: Playwright Java API Automation & Auth
categories: [Playwright Java API Automation & Auth, Playwright Java, Test Automation]
excerpt: >-
  Update backend resources using HTTP PUT (full replacement) and PATCH (partial update) in Playwright Java.
readTime: 9 min read
---

# PUT/PATCH APIs - Playwright Java API Automation & Auth

Updating existing REST resources requires using **HTTP PUT** for full resource replacements or **HTTP PATCH** for partial field updates.

Playwright Java provides dedicated `request.put()` and `request.patch()` methods.

---

## 1. PUT vs PATCH Execution

```java
// 1. HTTP PUT (Full Resource Update)
String fullUpdatePayload = "{\"name\": \"Pankaj Kumar\", \"email\": \"pankaj@mycodeyatra.com\", \"status\": \"Active\"}";
APIResponse putResponse = apiContext.put("/api/users/101", RequestOptions.create().setData(fullUpdatePayload));
assertThat(putResponse.status()).isEqualTo(200);
 
// 2. HTTP PATCH (Partial Resource Update)
String patchPayload = "{\"status\": \"Inactive\"}";
APIResponse patchResponse = apiContext.patch("/api/users/101", RequestOptions.create().setData(patchPayload));
assertThat(patchResponse.status()).isEqualTo(200);
```

---

## 2. Production API Client (`src/main/java/com/mycodeyatra/pages/api/PutPatchAPIsPage.java`)

```java
package com.mycodeyatra.pages.api;
 
import com.microsoft.playwright.APIRequestContext;
import com.microsoft.playwright.APIResponse;
import com.microsoft.playwright.options.RequestOptions;
 
public class PutPatchAPIsPage {
    private final APIRequestContext request;
 
    public PutPatchAPIsPage(APIRequestContext request) {
        this.request = request;
    }
 
    public APIResponse updateUser(int userId, String payload) {
        return request.put("/api/users/" + userId, RequestOptions.create().setData(payload));
    }
}
```

---

## 3. Key Takeaways

1. **PUT Replaces Entire Entity**: Missing fields in a PUT payload are typically set to `null` by backend servers.
2. **PATCH Modifies Specified Fields**: Only fields included in the PATCH body are updated.

