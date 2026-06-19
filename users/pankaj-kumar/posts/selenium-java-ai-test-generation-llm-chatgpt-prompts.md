---
title: The End of Manual Scripting: AI Test Generation in Java
date: 12-Oct-2026
lastUpdated: 12-Oct-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [java, selenium, ai, test-generation, chatgpt, llm, openai, prompt-engineering]
category: Selenium Java
categories: [Selenium Java, AI in Automation]
excerpt: >-
  Welcome to Phase 15. Learn how to use Prompt Engineering and Large Language Models (LLMs) to automatically generate perfect Page Object classes and TestNG scripts in seconds.
readTime: 6 min read
---

# The End of Manual Scripting: AI Test Generation in Java

For the last 14 Phases, we have meticulously built a massive, enterprise-grade Selenium Java framework from absolute scratch. 

We wrote every single Page Object class by hand. We wrote every single `By.xpath()` locator. We wrote every TestNG assertion. 

But the software industry is undergoing the largest shift since the invention of the internet. **Artificial Intelligence is changing everything.**

In this brand new phase, Phase 15, we are going to look to the future. In this tutorial, we will explore how Large Language Models (LLMs) like OpenAI's GPT-4, Google's Gemini, and Anthropic's Claude can literally write your Selenium Java test scripts for you.

---

## 1. The Traditional Workflow

Traditionally, an SDET's workflow looks like this:
1. Read the Jira Acceptance Criteria.
2. Open the Web Application.
3. Right-click and "Inspect Element" to find the ID or XPath of the username field.
4. Open IntelliJ IDEA.
5. Create a `LoginPage.java` file.
6. Write the `WebElement` declaration.
7. Write the `login()` method.
8. Create a `LoginTest.java` file.
9. Write the TestNG `@Test` method calling the Page Object.

This process takes about 20 to 45 minutes per scenario.

## 2. The AI Workflow (Prompt Engineering for SDETs)

What if you could skip steps 3 through 9 entirely?

With AI Test Generation, you provide an LLM with the HTML of the webpage and the Jira Acceptance Criteria, and it outputs perfectly formatted Page Object classes and TestNG scripts in seconds.

The secret to making this work is **Prompt Engineering**. You must tell the AI exactly what your framework architecture looks like.

### Example Prompt

```text
You are an expert Senior SDET specializing in Java and Selenium WebDriver.
I have a web application with the following HTML structure for the Login Page:
<form id="loginForm">
    <input type="text" id="user_email" name="email" class="form-control" />
    <input type="password" id="user_password" name="password" class="form-control" />
    <button type="submit" id="btn_login" class="btn btn-primary">Sign In</button>
</form>
<div id="error_message" class="alert-danger" style="display:none;"></div>
Task 1: Generate a Page Object Model class named `LoginPage.java`.
Rules:
- Do not use PageFactory (@FindBy).
- Use `By` locators at the top of the class.
- Include a constructor that accepts a `WebDriver`.
- Write a `login(String email, String password)` method that returns a new instance of `HomePage`.
- Write a `getErrorMessage()` method.
Task 2: Generate a TestNG class named `LoginTest.java`.
Rules:
- The class should extend `BaseTest`.
- Write a test method named `testInvalidLoginDisplaysErrorMessage()`.
- Assert that the error message equals "Invalid credentials".
```

### The AI's Output

Within seconds, the LLM will generate the exact code you need:

**LoginPage.java**

```java
import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;
public class LoginPage {
    private WebDriver driver;
    private By emailField = By.id("user_email");
    private By passwordField = By.id("user_password");
    private By loginButton = By.id("btn_login");
    private By errorMessage = By.id("error_message");
    public LoginPage(WebDriver driver) {
        this.driver = driver;
    }
    public HomePage login(String email, String password) {
        driver.findElement(emailField).sendKeys(email);
        driver.findElement(passwordField).sendKeys(password);
        driver.findElement(loginButton).click();
        return new HomePage(driver);
    }
    public String getErrorMessage() {
        return driver.findElement(errorMessage).getText();
    }
}
```

**LoginTest.java**

```java
import org.testng.Assert;
import org.testng.annotations.Test;
public class LoginTest extends BaseTest {
    @Test
    public void testInvalidLoginDisplaysErrorMessage() {
        LoginPage loginPage = new LoginPage(driver);
        loginPage.login("wrong@email.com", "badpassword");
        String actualError = loginPage.getErrorMessage();
        Assert.assertEquals(actualError, "Invalid credentials", "Error message did not match!");
    }
}
```

---

## 3. Automating the Prompt with Playwright/Selenium

Copying and pasting HTML from Chrome into ChatGPT is manual. We can automate the prompt generation itself!

You can write a simple Selenium script that navigates to a URL, extracts the `document.body.innerHTML`, and sends it to the OpenAI API via Java!

```java
import com.theokanning.openai.service.OpenAiService;
import com.theokanning.openai.completion.chat.ChatCompletionRequest;
import com.theokanning.openai.completion.chat.ChatMessage;
public class AIGenerator {
    public void generateTest(String url, String acceptanceCriteria) {
        // 1. Use Selenium to get the raw HTML
        driver.get(url);
        String rawHtml = driver.getPageSource();
        // 2. Build the AI Prompt
        String prompt = "Generate a Java Selenium Page Object and TestNG test for this HTML:\n\n" 
                        + rawHtml + "\n\nAcceptance Criteria:\n" + acceptanceCriteria;
        // 3. Send it to OpenAI!
        OpenAiService service = new OpenAiService("YOUR_API_KEY");
        ChatCompletionRequest request = ChatCompletionRequest.builder()
                .model("gpt-4-turbo")
                .messages(List.of(new ChatMessage("user", prompt)))
                .build();
        String generatedJavaCode = service.createChatCompletion(request).getChoices().get(0).getMessage().getContent();
        System.out.println("AI GENERATED CODE:\n" + generatedJavaCode);
    }
}
```

## Conclusion

We are moving away from an era where SDETs write test scripts. We are entering an era where **SDETs write the prompts that generate the test scripts**.

By leveraging AI, you can reduce the time it takes to build Page Objects from 45 minutes to 45 seconds. Your job shifts from "script writer" to "code reviewer and framework architect".

But test generation is only half the battle. The biggest problem in UI automation is maintenance: Locators constantly break when developers change the UI.

What if AI could fix broken locators for you automatically in the middle of a test execution? 

In our next tutorial, we will explore the magic of **AI Locator Healing**!
