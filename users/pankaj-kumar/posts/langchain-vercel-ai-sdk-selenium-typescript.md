---
title: Enterprise Orchestration: LangChain.js & Vercel AI SDK
date: 19-Feb-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [typescript, selenium, ai, langchain, vercel-ai-sdk, tool-calling, agents]
category: Selenium TypeScript
categories: [Selenium TypeScript, AI in Testing]
excerpt: >-
  Upgrade your Autonomous Agents to production-grade. Compare the architectural differences between LangChain.js and the Vercel AI SDK, and learn how to implement robust Tool Calling for your testing framework.
readTime: 4 min read
---

# Enterprise Orchestration: LangChain.js & Vercel AI SDK

In our previous tutorial, we constructed a primitive Autonomous Agent from scratch. We wrote a recursive function to manage the ReAct loop, manually parsed strings to extract tools and arguments, and fed the observations back into the LLM context.

While this is a fantastic learning exercise, it is not how you build AI software in production.

Just as you wouldn't write a web server from scratch in Node.js without Express, you shouldn't write an Autonomous Agent from scratch. You should use an orchestration framework. 

Today, we will explore the two titans of TypeScript AI orchestration: **LangChain.js** and the **Vercel AI SDK**.

---

## 1. The Challenge of Tool Calling

The hardest part of building an Autonomous Agent is **Tool Calling** (also known as Function Calling). 

If you want the AI to execute a Selenium test, you must:
1. Provide the AI with a JSON schema describing the `runSeleniumTest` function.
2. Wait for the AI to return a JSON object containing the arguments (e.g., `{"tag": "@login"}`).
3. Parse the JSON, handle syntax errors, execute the function, and return the result.

Frameworks like LangChain and Vercel AI SDK handle all of this boilerplate for you.

---

## 2. Using the Vercel AI SDK

The Vercel AI SDK is arguably the cleanest and most developer-friendly framework for building Agents in TypeScript. It provides a standardized `generateText` method that inherently supports Tool Calling.

Here is how you would expose your Selenium framework to an AI using the Vercel SDK:

```typescript
import { generateText, tool } from 'ai';
import { openai } from '@ai-sdk/openai';
import { z } from 'zod';
import { exec } from 'child_process';
async function runTestAgent() {
  const result = await generateText({
    model: openai('gpt-4o'),
    prompt: 'Check the current build status. If there are any recent changes, run the @smoke tests.',
    tools: {
      runSeleniumTest: tool({
        description: 'Executes a Selenium Cucumber test suite for a given tag.',
        parameters: z.object({
          tag: z.string().describe('The cucumber tag to run (e.g., @smoke)'),
        }),
        execute: async ({ tag }) => {
          return new Promise((resolve) => {
            console.log(`[AGENT] Executing test for tag: ${tag}`);
            exec(`npm run test -- --tags "${tag}"`, (error, stdout) => {
              resolve(error ? `Test Failed: ${stdout}` : `Test Passed: ${stdout}`);
            });
          });
        },
      }),
      // You can easily add more tools here!
      // checkBuildStatus: tool({...}),
      // updateJiraTicket: tool({...})
    },
    maxSteps: 5, // Important: Prevents infinite loops!
  });
  console.log('Final Agent Report:', result.text);
}
runTestAgent();
```

Notice the `maxSteps: 5` property. This is a critical safety feature. It tells the SDK to automatically handle the ReAct loop, feeding tool observations back to the AI, but it forces the Agent to stop if it hasn't solved the objective in 5 steps!

---

## 3. Using LangChain.js

While Vercel AI SDK is incredibly sleek, **LangChain.js** is the undisputed heavy-weight champion of enterprise AI. It provides a massive ecosystem of pre-built integrations, Vector Databases, and complex routing graphs (LangGraph).

LangChain's architecture revolves around the concept of **Agents** and **Executors**:

```typescript
import { ChatOpenAI } from "@langchain/openai";
import { AgentExecutor, createToolCallingAgent } from "langchain/agents";
import { DynamicTool } from "@langchain/core/tools";
import { ChatPromptTemplate } from "@langchain/core/prompts";
import { exec } from 'child_process';
// 1. Define the Tool
const runSeleniumTool = new DynamicTool({
  name: "run_selenium_test",
  description: "Executes a Selenium test for a specific tag.",
  func: async (tag: string) => {
    return new Promise((resolve) => {
      exec(`npm run test -- --tags "${tag}"`, (error, stdout) => {
        resolve(stdout);
      });
    });
  },
});
// 2. Initialize the Model and Prompt
const model = new ChatOpenAI({ modelName: "gpt-4o" });
const prompt = ChatPromptTemplate.fromMessages([
  ["system", "You are an autonomous QA Automation Engineer."],
  ["human", "{input}"],
  ["placeholder", "{agent_scratchpad}"] // This holds the memory of previous steps!
]);
// 3. Create the Agent and the Executor
const agent = createToolCallingAgent({ llm: model, tools: [runSeleniumTool], prompt });
const agentExecutor = new AgentExecutor({ agent, tools: [runSeleniumTool] });
// 4. Run the Agent
async function main() {
  const result = await agentExecutor.invoke({
    input: "Run the @regression tests."
  });
  console.log(result.output);
}
main();
```

---

## 4. Which Framework Should You Choose?

If you are building a lightweight AI script or integrating an Agent into a Next.js application, the **Vercel AI SDK** is the clear winner. Its API is modern, typesafe, and incredibly clean.

If you are building a massive, highly complex system that requires the AI to query databases, search PDF documents, maintain long-term memory across sessions, and orchestrate multiple sub-agents, **LangChain.js** is the industry standard.

## Conclusion

We have completely automated the Automation Engineer. By hooking our Selenium TypeScript framework into LangChain or the Vercel AI SDK, we have given our code a brain.

We are now at the very end of our journey. In the final tutorial of this curriculum, we will look back at everything we have built—from Phase 1 to Phase 15—and discuss the ultimate **Future of Test Automation**.
