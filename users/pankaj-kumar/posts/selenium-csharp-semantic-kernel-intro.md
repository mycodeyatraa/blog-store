---
title: "Introduction to Semantic Kernel for Test Automation"
date: "31-Jan-2025"
description: "Welcome to the future of testing! Learn how to integrate Large Language Models (LLMs) into your C# Selenium framework using Microsoft's Semantic Kernel."
categories: ["Selenium C#", "Test Automation", "AI"]
tags: ["Selenium", "C#", ".NET", "AI", "Semantic Kernel", "LLM", "OpenAI", "Framework Design"]
author: "Pankaj Kumar"
lastUpdated: "31-Jan-2025"
---

Welcome to the brand new series: **AI-Powered Testing with Semantic Kernel & C#**!

For the past decade, Test Automation has been strictly deterministic: *If X happens, assert Y*. But modern applications are increasingly dynamic, generating AI-driven text, varied layouts, and localized content. Traditional assertions fail brittlely when text isn't a 100% exact match.

To test intelligent applications, we need intelligent tests. Today, we break into the most advanced tier of automation engineering by embedding Large Language Models (LLMs) directly into our C# framework using **Microsoft's Semantic Kernel**.

---

## 1. What is Semantic Kernel?

[Semantic Kernel](https://learn.microsoft.com/en-us/semantic-kernel/overview/) is Microsoft's open-source, lightweight SDK that lets you easily mix conventional programming languages (like C#) with the latest Large Language Models (like OpenAI, Azure OpenAI, Hugging Face, etc.).

**Why use Semantic Kernel in Test Automation?**
1. **Intelligent Assertions**: Instead of `Assert.AreEqual()`, ask the LLM: *"Does this response successfully answer the user's question?"*
2. **Dynamic Test Data**: Generate infinite, edge-case test data on the fly.
3. **Self-Healing Tests**: Ask the LLM to analyze the DOM and suggest a new XPath when a test fails.
4. **Log Analysis**: Automatically summarize massive failure logs into plain English.

---

## 2. Installing Semantic Kernel

Let's add the core Semantic Kernel package to our C# project!

Open your terminal in the root of your test project and run:

```sh
dotnet add package Microsoft.SemanticKernel
```

---

## 3. Your First AI-Powered NUnit Test

To use Semantic Kernel, you need an API key from an LLM provider. In this example, we will use OpenAI. *(Remember: Never hardcode your API keys in your code! Use Environment Variables or a `.runsettings` file).*

Create a new class named `Blog30_IntroToSK.cs`:

```cs
using Microsoft.SemanticKernel;
using NUnit.Framework;
using System;
using System.Threading.Tasks;
namespace mcyt_sel_csharp.AITests
{
    [TestFixture]
    public class Blog30_IntroToSK
    {
        private Kernel _kernel;
        [SetUp]
        public void Setup()
        {
            // Retrieve your API key from Environment Variables
            string apiKey = Environment.GetEnvironmentVariable("OPENAI_API_KEY") 
                            ?? throw new Exception("API Key not found!");
            // Initialize the Semantic Kernel builder
            var builder = Kernel.CreateBuilder();
            // Configure it to use OpenAI's GPT-4o model
            builder.AddOpenAIChatCompletion(
                modelId: "gpt-4o",
                apiKey: apiKey
            );
            // Build the Kernel!
            _kernel = builder.Build();
        }
        [Test]
        public async Task Test_AskKernelAQuestion()
        {
            // Define a prompt
            string prompt = "Explain what Selenium WebDriver is in exactly one sentence.";
            // Invoke the LLM asynchronously
            var response = await _kernel.InvokePromptAsync(prompt);
            Console.WriteLine("AI Response: " + response.ToString());
            // Basic assertion to ensure we got a response
            Assert.That(response.ToString(), Is.Not.Null.And.Not.Empty);
        }
    }
}
```

### Breaking it down:
1. **`Kernel.CreateBuilder()`**: This is the orchestrator. It prepares the environment for our LLM.
2. **`AddOpenAIChatCompletion`**: We specify which model we want (`gpt-4o`) and provide our credentials. Semantic Kernel handles all the complex HTTP requests and JSON parsing behind the scenes!
3. **`InvokePromptAsync`**: We pass our English instruction directly to the LLM and await the response.

---

## 4. Running the Test

When you execute this test using `dotnet test`, the console output will look something like this:

```text
AI Response: Selenium WebDriver is an open-source tool that allows automated testing of web applications by directly controlling web browsers through various programming languages.
```

Just like that, you have successfully called an advanced AI model directly from your NUnit test suite! 

---

## Conclusion

We have taken our first monumental step into the future of testing. By integrating Semantic Kernel, our framework is no longer just a script execution engine—it is now a cognitive agent.

In our next blog, we will put this power to work by using the LLM to **dynamically generate complex test data** for our Data-Driven Tests on the fly!

Happy Automating!
