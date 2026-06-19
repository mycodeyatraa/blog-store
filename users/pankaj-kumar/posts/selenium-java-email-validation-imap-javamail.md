---
title: Automating the Inbox: Validating Emails with JavaMail API
date: 04-Aug-2026
lastUpdated: 04-Aug-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [java, selenium, email-validation, imap, javamail, test-automation, backend-validation]
category: Selenium Java
categories: [Selenium Java, Enterprise Validation]
excerpt: >-
  Stop automating the Gmail web UI. Learn how to integrate the JavaMail API into your Selenium framework to securely connect to IMAP servers, poll for asynchronous emails, and validate 2FA or Password Reset links natively.
readTime: 6 min read
---

# Automating the Inbox: Validating Emails with JavaMail API

So far in this series, we have mastered validating internal enterprise systems. We connected to MySQL, pulled NoSQL documents from MongoDB, parsed GraphQL APIs, and even intercepted real-time Kafka event streams.

But what happens when the system sends data *outside* the enterprise?

Almost every application relies on Email. Whether it is a "Welcome" email after registration, a "Password Reset" link, or an "Order Confirmation" receipt, verifying that these emails actually arrive in the user's inbox is a critical part of End-to-End testing.

In this tutorial, we will learn how to integrate the **JavaMail API** into our Selenium framework to connect to an external Inbox, read live emails, and assert their contents!

---

## 1. The Email Automation Architecture

You cannot (and should not) automate the Gmail or Outlook web UI using Selenium. Google actively blocks Selenium bots from logging into Gmail, and the UI changes constantly.

Instead, we will bypass the browser entirely and use the **IMAP (Internet Message Access Protocol)** to connect directly to the email server at the protocol level.

To do this, we need the official `javax.mail` dependency in our `pom.xml`.

```xml
<dependency>
    <groupId>com.sun.mail</groupId>
    <artifactId>javax.mail</artifactId>
    <version>1.6.2</version>
</dependency>
```

---

## 2. Using App Passwords for Testing

If you are using a Gmail account for testing, Google will block basic username/password logins via IMAP for security reasons. You must generate an **App Password**.

1. Go to your Google Account -> Security.
2. Enable 2-Step Verification.
3. Search for "App Passwords" and generate a 16-character password specifically for your automation framework.

*(Note: In enterprise environments, teams often use temporary email services like Mailtrap or Mailinator for testing instead of real Gmail accounts).*

---

## 3. Writing the Email Validation Test

Let's write an End-to-End test for a "Password Reset" workflow. 

We will use Selenium to click "Forgot Password" on the UI. Then, we will connect to the Gmail IMAP server, poll the inbox for the new email, extract the HTML content, and assert that the reset link exists!

```java
import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.testng.Assert;
import org.testng.annotations.AfterMethod;
import org.testng.annotations.BeforeMethod;
import org.testng.annotations.Test;
import javax.mail.*;
import javax.mail.search.FlagTerm;
import java.util.Properties;
public class EmailValidationTest {
    WebDriver driver;
    // Email Credentials
    String emailAddress = "automation.test.bot@gmail.com";
    String appPassword = "abcd efgh ijkl mnop"; // The 16-character Google App Password
    @BeforeMethod
    public void setup() {
        driver = new ChromeDriver();
    }
    @Test
    public void testPasswordResetEmailArrives() throws Exception {
        // --- PART 1: The UI Workflow ---
        driver.get("https://practice.mycodeyatra.com/forgot-password");
        driver.findElement(By.id("email-input")).sendKeys(emailAddress);
        driver.findElement(By.id("send-reset-link")).click();
        Assert.assertTrue(driver.findElement(By.id("success-toast")).isDisplayed());
        // --- PART 2: The IMAP Email Validation ---
        // 1. Configure the IMAP Properties for Gmail
        Properties props = new Properties();
        props.put("mail.store.protocol", "imaps");
        props.put("mail.imaps.host", "imap.gmail.com");
        props.put("mail.imaps.port", "993");
        props.put("mail.imaps.timeout", "10000");
        // 2. Connect to the Mail Store
        Session session = Session.getDefaultInstance(props, null);
        Store store = session.getStore("imaps");
        store.connect("imap.gmail.com", emailAddress, appPassword);
        // 3. Open the Inbox folder
        Folder inbox = store.getFolder("INBOX");
        inbox.open(Folder.READ_WRITE); // READ_WRITE so we can mark emails as Read!
        System.out.println("Connected to Gmail Inbox. Searching for new messages...");
        boolean emailFound = false;
        long endTime = System.currentTimeMillis() + 15000; // Poll for up to 15 seconds
        while (System.currentTimeMillis() < endTime) {
            // 4. Search ONLY for Unread emails (to optimize speed)
            Message[] messages = inbox.search(new FlagTerm(new Flags(Flags.Flag.SEEN), false));
            for (Message message : messages) {
                String subject = message.getSubject();
                // 5. Check if the subject matches our test
                if (subject != null && subject.contains("Password Reset Request")) {
                    emailFound = true;
                    // 6. Extract the Content
                    String content = message.getContent().toString();
                    System.out.println("Email Found! Content: " + content);
                    // 7. Assert the Reset Link exists in the email!
                    Assert.assertTrue(content.contains("https://practice.mycodeyatra.com/reset?token="), 
                        "The reset link is missing from the email body!");
                    // 8. Mark the email as "Read" so it doesn't break future test runs
                    message.setFlag(Flags.Flag.SEEN, true);
                    break;
                }
            }
            if (emailFound) break;
            Thread.sleep(2000); // Wait 2 seconds before checking again
        }
        // 9. Close the connections
        inbox.close(false);
        store.close();
        // 10. Final Assertion
        Assert.assertTrue(emailFound, "CRITICAL BUG: The Password Reset email never arrived in the inbox!");
    }
    @AfterMethod
    public void teardown() {
        driver.quit();
    }
}
```

---

## 4. Polling vs Sleeping

Just like our Kafka tutorial, Email delivery is highly asynchronous. When you click "Send Reset Link", the backend server puts the email into a queue (like AWS SES or SendGrid), which eventually routes it to Google's servers. This can take anywhere from 1 to 10 seconds.

If you simply write `Thread.sleep(10000)` before checking the inbox, your test suite will become unbearably slow. By writing a custom `while` loop that polls the inbox every 2 seconds, the test exits the exact millisecond the email arrives, keeping your CI/CD pipeline as fast as possible!

## Conclusion

Automating email verification separates novice automation engineers from Senior SDETs. Instead of building brittle, flaky UI scripts that attempt to click through the Gmail web interface, integrating the `javax.mail` API allows you to connect natively at the protocol level.

Whether you are extracting Two-Factor Authentication (2FA) OTP codes, clicking dynamic password reset links, or verifying PDF invoice attachments, the JavaMail API gives your Selenium framework absolute control over the inbox.

Speaking of PDF attachments, how do we automate those? In our next tutorial, we will learn how to parse and validate binary files using **PDF Validation**!
