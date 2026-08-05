---
id: "post-738"
title: "Email Validation in Playwright Java"
slug: "email-validation-in-playwright-java"
date: "21-Aug-2026"
author: "pankaj-kumar"
series: "playwright-java-enterprise-data"
seriesOrder: 7
topic: "7. Email & OTP Verification"
tags: ["Playwright", "Java", "Email", "Mailpit", "OTP"]
category: "Automation Testing"
categories: ["Automation Testing", "Playwright Java", "Email Testing"]
excerpt: "Automate registration verification links and OTP codes using Mailpit / Mailhog APIs combined with Playwright Java."
readTime: "8 min read"
---

# Email Validation in Playwright Java

End-to-end testing of user registration, password resets, and multi-factor authentication requires validating email delivery and capturing activation links or OTP codes.

---

## 1. Architectural Overview

Using mock SMTP servers like **Mailpit** or **Mailhog**, Playwright Java queries REST APIs to retrieve captured transactional emails, extract activation URLs or OTP codes, and proceed with browser automation.

```
+-----------------------------+       +-----------------------------+
| Playwright Web Script       | ----> | Frontend User Registration  |
+-----------------------------+       +-----------------------------+
               |                                     |
               | Query Rest API                      | Send Email (SMTP)
               v                                     v
+-------------------------------------------------------------------+
| Mock Mail Server (Mailpit / Mailhog on Port 1025 / 8025)          |
+-------------------------------------------------------------------+
```

---

## 2. Mailpit Client Utility

```java
// src/main/java/com/mycodeyatra/email/MailpitClient.java
package com.mycodeyatra.email;
 
import com.microsoft.playwright.APIRequestContext;
import com.microsoft.playwright.APIResponse;
import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.ObjectMapper;
 
import java.io.IOException;
import java.util.regex.Matcher;
import java.util.regex.Pattern;
 
public class MailpitClient {
    private final APIRequestContext request;
    private final ObjectMapper mapper = new ObjectMapper();
 
    public MailpitClient(APIRequestContext request) {
        this.request = request;
    }
 
    public String getActivationLink(String recipientEmail) throws IOException {
        APIResponse response = request.get("http://localhost:8025/api/v1/messages");
        JsonNode root = mapper.readTree(response.text());
        JsonNode messages = root.get("messages");
 
        for (JsonNode msg : messages) {
            JsonNode toArray = msg.get("To");
            for (JsonNode to : toArray) {
                if (to.get("Address").asText().equalsIgnoreCase(recipientEmail)) {
                    String msgId = msg.get("ID").asText();
                    return extractLinkFromMessage(msgId);
                }
            }
        }
        return null;
    }
 
    private String extractLinkFromMessage(String msgId) throws IOException {
        APIResponse response = request.get("http://localhost:8025/api/v1/message/" + msgId);
        JsonNode msgDetail = mapper.readTree(response.text());
        String body = msgDetail.get("Text").asText();
 
        Matcher matcher = Pattern.compile("https?://\S+/activate\?code=\w+").matcher(body);
        if (matcher.find()) {
            return matcher.group(0);
        }
        return null;
    }
}
```

---

## 3. Playwright Email Activation Test

```java
// src/test/java/com/mycodeyatra/tests/EmailVerificationTest.java
package com.mycodeyatra.tests;
 
import com.microsoft.playwright.*;
import com.mycodeyatra.email.MailpitClient;
import org.junit.jupiter.api.*;
 
import java.io.IOException;
 
import static org.junit.jupiter.api.Assertions.*;
 
public class EmailVerificationTest {
    private static Playwright playwright;
    private static Browser browser;
    private Page page;
 
    @BeforeAll
    static void setup() {
        playwright = Playwright.create();
        browser = playwright.chromium().launch(new BrowserType.LaunchOptions().setHeadless(true));
    }
 
    @BeforeEach
    void init() {
        page = browser.newPage();
    }
 
    @Test
    @DisplayName("Complete Registration Flow via Email Verification Link")
    void testEmailActivationFlow() throws IOException {
        String email = "qa_user_" + System.currentTimeMillis() + "@mycodeyatra.com";
 
        // 1. Submit Registration Form
        page.navigate("https://mycodeyatra.com/signup");
        page.fill("#email", email);
        page.click("#signup-btn");
 
        // 2. Extract activation link using Mailpit REST API
        MailpitClient mailClient = new MailpitClient(playwright.request().newContext());
        String activationUrl = mailClient.getActivationLink(email);
        assertNotNull(activationUrl, "Activation link should be found in Mailpit inbox");
 
        // 3. Navigate to extracted activation link via Playwright
        page.navigate(activationUrl);
        assertTrue(page.isVisible("#account-activated-msg"));
    }
 
    @AfterAll
    static void tearDown() {
        if (browser != null) browser.close();
        if (playwright != null) playwright.close();
    }
}
```

---

## 4. Enterprise Best Practices & Comparison Summary

- **Isolated Inboxes**: Use dynamic unique email prefixes to avoid cross-test pollution in shared email servers.
- **REST API Extraction**: Fetching message content via HTTP APIs is faster and less prone to UI flakiness than automating webmail UIs.
