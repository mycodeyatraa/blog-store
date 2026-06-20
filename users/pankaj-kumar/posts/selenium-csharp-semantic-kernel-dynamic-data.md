---
title: "Generating Dynamic Test Data with LLMs in C#"
date: "01-Feb-2025"
description: "Escape the limitations of static test data! Learn how to use Semantic Kernel and OpenAI to generate thousands of dynamic, edge-case testing permutations on the fly."
categories: ["Selenium C#", "Test Automation", "AI"]
tags: ["Selenium", "C#", ".NET", "AI", "Semantic Kernel", "Data-Driven Testing", "OpenAI"]
author: "Pankaj Kumar"
lastUpdated: "01-Feb-2025"
---

Welcome back to the **AI-Powered Testing** series!

In our Data-Driven Testing blogs, we learned how to pass data into our Selenium scripts using `.json` files and Excel spreadsheets. While incredibly useful, static data has a fatal flaw: **It is static**. 

If you hardcode 5 negative email test cases in an Excel file, your suite will only ever test those exact 5 emails. It will never randomly discover the 6th edge case that crashes your production server.

Today, we will use **Semantic Kernel** to ask an LLM to dynamically generate realistic, malicious, and unexpected edge cases on the fly, directly inside our NUnit test!

---

## 1. The Power of Prompt Engineering for Data

Large Language Models like GPT-4 are essentially massive databases of human knowledge. They know exactly what "valid" and "invalid" email formats look like, including obscure RFC specifications.

Instead of writing a C# generator, we simply ask the AI:
> *"Generate 5 highly complex, invalid email addresses that are designed to break a registration form. Return them as a comma-separated list without any extra text."*

---

## 2. Implementing the AI Data Generator

Let's create a new test file named `Blog31_AIDataGeneration.cs` in our `AITests` folder.

```cs
using Microsoft.SemanticKernel;
using NUnit.Framework;
using System;
using System.Threading.Tasks;
using System.Linq;
namespace mcyt_sel_csharp.AITests
{
    [TestFixture]
    public class Blog31_AIDataGeneration
    {
        private Kernel _kernel;
        [SetUp]
        public void Setup()
        {
            string apiKey = Environment.GetEnvironmentVariable("OPENAI_API_KEY") 
                            ?? throw new Exception("API Key not found!");
            var builder = Kernel.CreateBuilder();
            builder.AddOpenAIChatCompletion("gpt-4o", apiKey);
            _kernel = builder.Build();
        }
        [Test]
        public async Task Test_DynamicNegativeEmailValidation()
        {
            // 1. Ask the AI to generate our test data!
            string prompt = @"Generate 5 highly complex, invalid email addresses 
                              designed to break a UI registration form. 
                              Return ONLY a comma-separated list. Do not include spaces or extra text.";
            var result = await _kernel.InvokePromptAsync(prompt);
            string aiResponse = result.ToString();
            // 2. Parse the AI's response into a C# Array
            string[] invalidEmails = aiResponse.Split(new[] { ',' }, StringSplitOptions.RemoveEmptyEntries);
            Console.WriteLine($"AI Generated {invalidEmails.Length} Invalid Emails:");
            // 3. Execute our Data-Driven Test dynamically!
            foreach (var email in invalidEmails)
            {
                Console.WriteLine($"Testing Email: {email}");
                // In a real framework, you would drive Selenium here:
                // _loginPage.EnterEmail(email);
                // _loginPage.ClickSubmit();
                // Assert.IsTrue(_loginPage.IsErrorMessageDisplayed());
                Assert.That(email, Does.Not.Contain(" "), "AI failed instructions, included spaces.");
            }
        }
    }
}
```

### Breaking it down:
1. **The Prompt**: The prompt explicitly commands the AI to return data in a specific, machine-readable format (`"ONLY a comma-separated list"`). This is critical, otherwise the AI will output conversational text like *"Here are 5 emails for you:"* which breaks our C# parser!
2. **Dynamic Execution**: We `Split` the AI's response into a string array, and immediately feed it into a `foreach` loop. 
3. **Infinite Coverage**: Every time this test runs in your CI/CD pipeline, the AI might generate slightly different, malicious variations, significantly increasing your test coverage over time!

---

## 3. Advanced Use Cases

Comma-separated lists are great, but what if you need an entire mock database payload?

Because Semantic Kernel is so powerful, you can ask the LLM to return complex **JSON objects**:

```cs
string prompt = @"Generate a JSON array containing 3 mock user objects. 
                  Each object must have 'FirstName' (Arabic characters), 'LastName' (Japanese characters), 
                  and 'Age' (integer > 100). Return ONLY raw JSON.";
```

You can then use `System.Text.Json.JsonSerializer.Deserialize<List<UserModel>>(aiResponse)` to instantly convert the AI's output into strongly-typed C# objects, and pass them directly into your Page Object Models!

---

## Conclusion

By using Semantic Kernel to generate test data, we shift from *Deterministic Testing* (where we test only what we explicitly hardcode) to *Exploratory Automated Testing*. Our tests are now capable of thinking of edge cases that we, the engineers, may have completely forgotten about!

In our next blog, we will take AI integration even further by teaching Semantic Kernel how to analyze the HTML DOM and **Self-Heal** broken XPaths when the UI changes!

Happy Automating!
