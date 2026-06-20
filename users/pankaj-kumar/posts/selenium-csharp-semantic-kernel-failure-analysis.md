---
title: "Analyzing Test Failures with Semantic Kernel & OpenAI"
date: "03-Feb-2025"
description: "Stop digging through massive stack traces! Learn how to use AI to automatically summarize C# Selenium test failures into clear, human-readable English."
categories: ["Selenium C#", "Test Automation", "AI"]
tags: ["Selenium", "C#", ".NET", "AI", "Semantic Kernel", "Log Analysis", "OpenAI"]
author: "Pankaj Kumar"
lastUpdated: "03-Feb-2025"
---

Welcome back to the **AI-Powered Testing** series!

If you run a suite of 500 automated tests overnight, chances are you'll wake up to a few failures. As an SDET, you have to dig through Jenkins/GitHub Actions logs, parse massive C# stack traces, and figure out *why* the test failed. 

Worse yet, if a Product Manager asks why the build is broken, you can't just hand them a NullReferenceException stack trace. You have to translate it into plain English.

Today, we will automate the translation process. We're going to teach **Semantic Kernel** how to read NUnit failure logs and generate a simple, 1-sentence executive summary explaining exactly what went wrong in the UI!

---

## 1. Intercepting Failures in NUnit

To analyze a failure, we first need to capture it. In NUnit, we do this using the `[TearDown]` attribute. When a test finishes (whether it passes or fails), the TearDown method runs. 

We can inspect `TestContext.CurrentContext.Result.Outcome` to see if it failed, and grab the raw Stack Trace!

---

## 2. Building the AI Log Analyzer

Let's create a new class in our `AITests` folder named `Blog33_FailureAnalyzer.cs`.

```cs
using Microsoft.SemanticKernel;
using NUnit.Framework;
using System;
using System.Threading.Tasks;
namespace mcyt_sel_csharp.AITests
{
    [TestFixture]
    public class Blog33_FailureAnalyzer
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
        public void Test_DeliberateFailureToTriggerAI()
        {
            // We deliberately throw an ugly, technical exception here to simulate a real failure!
            throw new OpenQA.Selenium.ElementNotInteractableException(
                "Element <button id='submit-btn' class='hidden'> is not reachable by keyboard");
        }
        [TearDown]
        public async Task AnalyzeFailureAsync()
        {
            // 1. Check if the test actually failed
            var testStatus = TestContext.CurrentContext.Result.Outcome.Status;
            if (testStatus == NUnit.Framework.Interfaces.TestStatus.Failed)
            {
                // 2. Grab the ugly technical logs
                string errorMessage = TestContext.CurrentContext.Result.Message;
                string stackTrace = TestContext.CurrentContext.Result.StackTrace;
                Console.WriteLine("--- ORIGINAL STACK TRACE ---");
                Console.WriteLine(errorMessage);
                Console.WriteLine(stackTrace);
                Console.WriteLine("----------------------------");
                // 3. Ask the AI to translate it for a human!
                string prompt = $@"
                    You are an expert QA Automation Engineer.
                    A Selenium C# test just failed with the following error and stack trace.
                    Please analyze it and explain what went wrong in EXACTLY one simple, non-technical sentence 
                    that a Product Manager would understand. Do not include coding jargon.
                    Error Message: {errorMessage}
                    Stack Trace: {stackTrace}
                ";
                var result = await _kernel.InvokePromptAsync(prompt);
                Console.WriteLine("\n--- 🤖 AI FAILURE SUMMARY ---");
                Console.WriteLine(result.ToString());
                Console.WriteLine("------------------------------");
                // In a real framework, you would push this 'result.ToString()' to a Slack or Teams webhook!
            }
        }
    }
}
```

### How it Works:
1. The test `Test_DeliberateFailureToTriggerAI` intentionally throws an `ElementNotInteractableException`. 
2. NUnit marks the test as **Failed** and immediately invokes `[TearDown]`.
3. We extract `TestContext.CurrentContext.Result.Message` which contains the ugly Selenium error string.
4. We feed this string into Semantic Kernel with a highly constrained persona prompt: *"Explain this in exactly one simple, non-technical sentence that a Product Manager would understand."*

---

## 3. The Result

When you run this test, instead of just getting a red X and a wall of text, the console prints:

```text
--- 🤖 AI FAILURE SUMMARY ---
The automated test failed because the submit button on the webpage was hidden and could not be clicked.
```

### The Enterprise Impact
Imagine tying this AI Summary into the ExtentReports framework we built back in Blog 7, or pushing this directly to a Slack Webhook! 

When a test fails overnight, the QA team no longer has to spend 20 minutes triaging the logs to see if it's an environment issue, a locator issue, or a real bug. The AI reads the stack trace and posts a perfect, human-readable summary right into your team's chat!

---

## Conclusion

By hooking Semantic Kernel into our NUnit `[TearDown]` lifecycle, we have essentially hired a junior QA engineer to sit and read our log files 24/7. 

In our next blog, we are going to combine everything we've learned about Semantic Kernel to build an **AI Test Assistant Chatbot** that allows you to trigger test executions via a conversational interface!

Happy Automating!
