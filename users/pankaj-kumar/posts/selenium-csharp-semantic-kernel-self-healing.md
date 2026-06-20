---
title: "Self-Healing Tests: Locating Elements via AI in Selenium C#"
date: "02-Feb-2025"
description: "Discover the holy grail of test automation: Self-Healing tests! Learn how to use Semantic Kernel to automatically find new XPaths when the UI changes."
categories: ["Selenium C#", "Test Automation", "AI"]
tags: ["Selenium", "C#", ".NET", "AI", "Semantic Kernel", "Self-Healing", "XPath", "OpenAI"]
author: "Pankaj Kumar"
lastUpdated: "02-Feb-2025"
---

Welcome back to the **AI-Powered Testing** series!

If you've been working in Test Automation for more than a week, you know the absolute worst enemy of any SDET: `NoSuchElementException`.

UI Developers constantly change IDs, Classes, and DOM structures. When they do, your hardcoded `By.Id("loginBtn")` breaks, and the pipeline turns red. Fixing broken locators accounts for roughly 60% of all automation maintenance time.

But what if your test could fix itself? Today, we will build a **Self-Healing Extension Method** using Semantic Kernel that automatically asks an AI for a new locator when the original one fails!

---

## 1. The Self-Healing Workflow

The concept of a Self-Healing test is incredibly elegant:
1. Try to find the element using the original locator.
2. If it succeeds, proceed normally (Zero overhead).
3. If it throws a `NoSuchElementException`, **catch the exception**.
4. Extract the `driver.PageSource` (the raw HTML DOM).
5. Send the HTML and the name of the target element to Semantic Kernel.
6. The LLM analyzes the DOM, finds the *new* element, and returns a working XPath!
7. The test uses the new XPath to complete the action!

---

## 2. Building the AI Locator Engine

Create a new file in your `AITests` folder named `Blog32_SelfHealingTests.cs`.

```cs
using Microsoft.SemanticKernel;
using NUnit.Framework;
using OpenQA.Selenium;
using OpenQA.Selenium.Chrome;
using System;
using System.Threading.Tasks;
namespace mcyt_sel_csharp.AITests
{
    [TestFixture]
    public class Blog32_SelfHealingTests
    {
        private IWebDriver _driver;
        private Kernel _kernel;
        [SetUp]
        public void Setup()
        {
            _driver = new ChromeDriver();
            string apiKey = Environment.GetEnvironmentVariable("OPENAI_API_KEY") 
                            ?? throw new Exception("API Key not found!");
            var builder = Kernel.CreateBuilder();
            builder.AddOpenAIChatCompletion("gpt-4o", apiKey);
            _kernel = builder.Build();
        }
        [TearDown]
        public void Teardown()
        {
            _driver.Quit();
            _driver.Dispose();
        }
        // The Magic Self-Healing Method
        public async Task<IWebElement> FindElementWithHealingAsync(By originalLocator, string elementDescription)
        {
            try
            {
                // 1. Attempt standard fast lookup
                return _driver.FindElement(originalLocator);
            }
            catch (NoSuchElementException)
            {
                Console.WriteLine($"[HEALING] Locator {originalLocator} failed. Asking AI to find '{elementDescription}'...");
                // 2. Extract DOM (Truncated for token limits in real life!)
                string pageSource = _driver.PageSource;
                // 3. Build Prompt
                string prompt = $@"
                    I am trying to find a web element described as '{elementDescription}'.
                    The original locator failed because the UI changed.
                    Analyze this HTML snippet and return ONLY the most robust XPath to find it.
                    Do NOT include markdown backticks or explanations. Just the raw XPath string.
                    HTML:
                    {pageSource.Substring(0, Math.Min(pageSource.Length, 3000))} 
                "; // Note: We substring the DOM to save tokens!
                // 4. Ask Semantic Kernel
                var result = await _kernel.InvokePromptAsync(prompt);
                string newXPath = result.ToString().Trim();
                Console.WriteLine($"[HEALING] AI suggested new XPath: {newXPath}");
                // 5. Try the AI's suggestion!
                return _driver.FindElement(By.XPath(newXPath));
            }
        }
        [Test]
        public async Task Test_SelfHealingLoginButton()
        {
            _driver.Navigate().GoToUrl("https://mycodeyatra.com/practice/login");
            // Deliberately providing a BROKEN locator to trigger the healing!
            By brokenLocator = By.Id("this-id-does-not-exist");
            // Instead of crashing, it will ask the AI to find the 'Login Button'
            IWebElement btn = await FindElementWithHealingAsync(brokenLocator, "The primary Submit Login button");
            // Click it using the AI-discovered element!
            btn.Click();
            Assert.Pass("The test healed itself and clicked the button successfully!");
        }
    }
}
```

### Critical Architectural Notes:
- **Token Limits**: Modern web pages have massive DOMs. Passing the entire `driver.PageSource` to an LLM will consume thousands of tokens (which costs money and time). In a production framework, you should use `driver.ExecuteScript` to extract only the HTML of the specific form or section you care about, rather than the whole `<body>`.
- **Systematic Updates**: In an enterprise system, your Self-Healing method should also write the newly discovered XPath into a log file or Slack channel so a developer can permanently update the Page Object Model later!

---

## Conclusion

We just completely neutralized `NoSuchElementException`! By hooking Semantic Kernel into our standard WebDriver flow, our automation framework is now resilient against sudden UI changes. 

While LLM inference takes a few seconds, a 5-second self-healing pause is infinitely better than a completely failed CI/CD pipeline blocking a release!

In our next blog, we will tackle the second most annoying part of automation: **Analyzing massive, incomprehensible failure logs**. We'll teach the AI to read our logs and tell us exactly *why* a test failed in plain English!

Happy Automating!
