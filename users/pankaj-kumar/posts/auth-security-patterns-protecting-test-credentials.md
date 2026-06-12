---
title: Auth Security Patterns: Protecting Test Credentials
date: 22-Sep-2024
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [selenium, java, security-testing, credentials, aws-secrets-manager, dotenv]
category: UI Automation
categories: [UI Automation, Selenium, Java]
excerpt: >-
  Stop leaking passwords to GitHub. Learn how to securely manage automation credentials using local .env files and cloud vaults like AWS Secrets Manager in your CI/CD pipelines.
readTime: 5 min read
---

# Auth Security Patterns: Protecting Test Credentials

We have successfully integrated OWASP ZAP, validated CSPs, and cryptographically verified our JWTs. But as we wrap up **Phase 7: Advanced Security Testing**, we must turn the spotlight inward and look at the security of our own automation framework.

The absolute fastest way for a hacker to breach your company is by finding a hardcoded password pushed to a GitHub repository by an SDET.

In this final security article, we will explore Enterprise Auth Security Patterns—learning how to completely decouple credentials from code by leveraging `.env` files locally and secure vault services (like AWS Secrets Manager) in CI/CD pipelines.

---

## 1. The Anti-Pattern: Hardcoded Secrets

A frightening amount of legacy automation code looks like this:

```java
// DANGEROUS ANTI-PATTERN
public class LoginTest {
    @Test
    public void testLogin() {
        driver.findElement(By.id("username")).sendKeys("admin@company.com");
        driver.findElement(By.id("password")).sendKeys("SuperSecretPass123!");
    }
}
```

Once that code is pushed to GitHub or GitLab, the password is permanently stored in the commit history. Even if you delete it later, bots actively scrape repositories for exposed secrets.

---

## 2. Local Execution: The `.env` Pattern

For developers running tests locally, the industry standard is to use a `.env` file. This file sits on your local machine, contains all the passwords, and is explicitly ignored by `git` using a `.gitignore` file.

To read `.env` files in Java, add the `dotenv-java` library to your `pom.xml`:

```xml
<dependency>
    <groupId>io.github.cdimascio</groupId>
    <artifactId>dotenv-java</artifactId>
    <version>3.0.0</version>
</dependency>
```

**Step A:** Create `.env` in the root of your project:
```env
# Do NOT commit this file!
QA_ADMIN_USER=admin@company.com
QA_ADMIN_PASS=SuperSecretPass123!
```

**Step B:** Update your Java configuration class:
```java
package com.mycodeyatra.config;
 
import io.github.cdimascio.dotenv.Dotenv;
 
public class ConfigReader {
    private static final Dotenv dotenv = Dotenv.load();
 
    public static String getAdminUsername() {
        return dotenv.get("QA_ADMIN_USER");
    }
 
    public static String getAdminPassword() {
        return dotenv.get("QA_ADMIN_PASS");
    }
}
```

Now, your test becomes:
```java
driver.findElement(By.id("password")).sendKeys(ConfigReader.getAdminPassword());
```

---

## 3. CI/CD Execution: The Secrets Manager Pattern

While `.env` files are great locally, you cannot use them in a cloud CI/CD pipeline (like Jenkins, GitHub Actions, or GitLab CI) without manually placing the file on the build server.

Enterprise companies use **Secure Vaults** like AWS Secrets Manager, Azure Key Vault, or HashiCorp Vault. At runtime, your Java test reaches out to AWS, requests the password, and uses it entirely in memory.

Here is how you fetch a password from AWS Secrets Manager using the AWS SDK for Java:

```java
package com.mycodeyatra.security;
 
import software.amazon.awssdk.regions.Region;
import software.amazon.awssdk.services.secretsmanager.SecretsManagerClient;
import software.amazon.awssdk.services.secretsmanager.model.GetSecretValueRequest;
import software.amazon.awssdk.services.secretsmanager.model.GetSecretValueResponse;
import org.json.JSONObject;
 
public class AwsSecretsManagerUtils {
 
    public static String getSecret(String secretName, String key) {
 
        Region region = Region.US_EAST_1;
        SecretsManagerClient client = SecretsManagerClient.builder()
                .region(region)
                .build();
 
        GetSecretValueRequest getSecretValueRequest = GetSecretValueRequest.builder()
                .secretId(secretName)
                .build();
 
        GetSecretValueResponse getSecretValueResponse;
 
        try {
            getSecretValueResponse = client.getSecretValue(getSecretValueRequest);
        } catch (Exception e) {
            throw new RuntimeException("Failed to fetch secret from AWS", e);
        }
 
        String secretString = getSecretValueResponse.secretString();
 
        // AWS stores secrets as JSON strings. Parse it to get the specific key.
        JSONObject jsonObject = new JSONObject(secretString);
        return jsonObject.getString(key);
    }
}
```

---

## 4. Building an Intelligent Configuration Factory

We don't want to change our code every time we switch between running locally vs in CI. We can build an intelligent Configuration Factory that checks if it's running in CI; if so, it queries AWS, otherwise, it reads the `.env` file!

```java
public class SecureCredentialProvider {
 
    public static String getAdminPassword() {
        String env = System.getProperty("executionEnv", "local");
 
        if (env.equalsIgnoreCase("ci")) {
            // Fetch from AWS if running in Jenkins
            return AwsSecretsManagerUtils.getSecret("QA_Env_Secrets", "adminPassword");
        } else {
            // Fetch from .env if running on a developer's laptop
            return ConfigReader.getAdminPassword();
        }
    }
}
```

---

## System Architecture

Here is the secure data flow protecting your automation credentials:

![diagram_1](https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/posts/auth-security-patterns-protecting-test-credentials/images/diagram_1.png)

## Conclusion

Securing your framework is just as important as securing the application you are testing. By completely removing hardcoded credentials from your Git repository and intelligently switching between local `.env` files and AWS Secrets Manager, you eliminate the risk of catastrophic credential leaks.

This officially wraps up **Phase 7: Advanced Security Testing**! Next, we will shift gears dramatically as we enter **Phase 8: Visual Testing**, where we will explore pixel-perfect image comparison using AShot and AI-driven visual validation using Applitools!
