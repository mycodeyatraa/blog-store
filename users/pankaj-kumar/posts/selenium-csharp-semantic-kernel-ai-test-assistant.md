---
title: "Building an AI Test Assistant with Semantic Kernel Chatbots"
date: "04-Feb-2025"
description: "Learn how to build a conversational AI Test Assistant in C# using Semantic Kernel's ChatHistory to answer QA questions, generate test cases, and explain code!"
categories: ["Selenium C#", "Test Automation", "AI"]
tags: ["Selenium", "C#", ".NET", "AI", "Semantic Kernel", "Chatbot", "OpenAI"]
author: "Pankaj Kumar"
lastUpdated: "04-Feb-2025"
---

Welcome back to the **AI-Powered Testing** series!

So far, we have used Semantic Kernel in the background: dynamically generating data, self-healing broken XPaths, and analyzing stack traces. But what if we brought the AI to the foreground?

Imagine a Junior QA Engineer joins your team. Instead of asking you 50 questions a day about how your framework works, what if they could ask an AI Chatbot that is explicitly trained on your testing standards?

Today, we will build an interactive **AI Test Assistant** in C# using Semantic Kernel's `ChatHistory` functionality!

---

## 1. Prompts vs. Chat History

In our previous blogs, we used `InvokePromptAsync()`. This is great for "One-Shot" tasks. The AI reads the prompt, answers, and immediately forgets the conversation.

A Chatbot, however, requires **Memory**. When you ask a follow-up question, the AI needs to remember what it said 5 seconds ago. Semantic Kernel manages this state elegantly using the `ChatHistory` object and the `IChatCompletionService`.

---

## 2. Building the Chatbot

Let's create a new console application script (or a standalone Test class for demonstration) named `Blog34_TestAssistant.cs` inside our `AITests` folder.

```cs
using Microsoft.SemanticKernel;
using Microsoft.SemanticKernel.ChatCompletion;
using NUnit.Framework;
using System;
using System.Threading.Tasks;
namespace mcyt_sel_csharp.AITests
{
    [TestFixture]
    public class Blog34_TestAssistant
    {
        private Kernel _kernel;
        private IChatCompletionService _chatService;
        private ChatHistory _history;
        [SetUp]
        public void Setup()
        {
            string apiKey = Environment.GetEnvironmentVariable("OPENAI_API_KEY") 
                            ?? throw new Exception("API Key not found!");
            var builder = Kernel.CreateBuilder();
            builder.AddOpenAIChatCompletion("gpt-4o", apiKey);
            _kernel = builder.Build();
            // 1. Retrieve the Chat Service from the Kernel
            _chatService = _kernel.GetRequiredService<IChatCompletionService>();
            // 2. Initialize the Chat History with a System Prompt
            _history = new ChatHistory(
                "You are an expert QA Automation Architect. Your job is to help junior testers " +
                "understand Selenium C#, NUnit, and BDD. Keep answers concise, friendly, and include C# code snippets."
            );
        }
        [Test]
        public async Task Test_SimulateConversation()
        {
            // Turn 1: Junior Tester asks a question
            string userQuestion1 = "How do I click a button in Selenium C#?";
            Console.WriteLine($"[USER]: {userQuestion1}");
            _history.AddUserMessage(userQuestion1); // Add to memory
            var response1 = await _chatService.GetChatMessageContentAsync(_history, kernel: _kernel);
            _history.AddAssistantMessage(response1.Content); // Save AI's response to memory
            Console.WriteLine($"[AI BOT]: {response1.Content}\n");
            // Turn 2: Follow-up question (Testing memory!)
            string userQuestion2 = "Can you make it wait for the button first?";
            Console.WriteLine($"[USER]: {userQuestion2}");
            _history.AddUserMessage(userQuestion2);
            var response2 = await _chatService.GetChatMessageContentAsync(_history, kernel: _kernel);
            _history.AddAssistantMessage(response2.Content);
            Console.WriteLine($"[AI BOT]: {response2.Content}");
            Assert.Pass("Conversation completed successfully!");
        }
    }
}
```

### Breaking it Down:
1. **The System Prompt**: We initialized the `ChatHistory` with a System Message: *"You are an expert QA Automation Architect..."*. This forces the AI to adopt a specific persona, ensuring it doesn't start answering questions about cooking recipes!
2. **Managing State**: Every time the User asks a question, we call `_history.AddUserMessage()`. Every time the AI answers, we call `_history.AddAssistantMessage()`. 
3. **Contextual Awareness**: Notice the second question: *"Can you make it wait for the button first?"* The user doesn't specify *which* button or *what language* they are using. The AI knows exactly what they mean because the `ChatHistory` provides the context of the previous C# Selenium question!

---

## 3. Real-World Applications

While we executed this inside an NUnit test for demonstration, in the real world, you would wrap this Semantic Kernel logic inside a minimal Web API (using ASP.NET Core) and attach a simple React Frontend to it. 

You could then integrate this Chatbot directly into your internal company portal!

Furthermore, using Semantic Kernel **Plugins**, you could literally grant this Chatbot the ability to execute C# methods. A tester could type *"Run the Smoke Suite"*, and the AI would automatically trigger a Jenkins API call to run the tests!

---

## Conclusion

By leveraging Semantic Kernel's `IChatCompletionService` and `ChatHistory`, we transitioned from single-prompt scripts to stateful, conversational AI assistants. Your framework is no longer just a tool; it's a mentor.

In our **next and final blog of the year**, we will zoom out and discuss the **Future of Automation: C#, Selenium & AI**, analyzing where the industry is heading and how to stay ahead of the curve!

Happy Automating!
