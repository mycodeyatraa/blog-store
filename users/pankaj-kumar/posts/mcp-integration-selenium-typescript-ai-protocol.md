---
title: Unlocking the Protocol: MCP Integration
date: 17-Feb-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [typescript, selenium, ai, mcp, model-context-protocol, automation, llms]
category: Selenium TypeScript
categories: [Selenium TypeScript, AI in Testing]
excerpt: >-
  Standardize your AI infrastructure. Learn how to build a Model Context Protocol (MCP) Server in TypeScript, exposing your Selenium framework as a callable Tool for autonomous AI agents.
readTime: 4 min read
---

# Unlocking the Protocol: MCP Integration

In the previous tutorial, we built an AI Self-Healing mechanism. To make it work, our TypeScript code had to manually extract the HTML of the page, formulate a prompt string, and send a raw HTTP POST request to an LLM provider (like OpenAI or Anthropic).

This works, but it is incredibly brittle. 

What if the LLM needs to check a Jira ticket to see if the UI change was intentional? What if the LLM needs to query our Elasticsearch logs?

Instead of writing custom API wrappers for every single tool, the AI industry has created a universal standard: the **Model Context Protocol (MCP)**.

---

## 1. What is the Model Context Protocol?

The Model Context Protocol (MCP) is an open-source architecture that allows AI Assistants (like Claude or custom autonomous agents) to securely connect to external data sources and tools.

Instead of your automation framework calling the AI, **the AI calls your automation framework.**

By running an MCP Server on your local machine or in your CI/CD pipeline, you expose your databases, filesystems, and WebDriver instances to an LLM using a standardized JSON-RPC communication layer.

---

## 2. The Architecture of MCP

An MCP integration consists of three parts:

1. **The MCP Host (The AI):** The LLM that wants to perform an action (e.g., Anthropic Claude).
2. **The MCP Client:** The bridge application that communicates between the Host and the Server.
3. **The MCP Server:** A local Node.js or Python application that exposes specific **Resources** and **Tools** to the Client.

### Resources vs. Tools
- **Resources:** Read-only data. E.g., The MCP Server exposes the `allure-results` JSON files so the AI can read why a test failed.
- **Tools:** Executable functions. E.g., The MCP Server exposes a `run_cucumber_test` function. The AI can actually command your machine to start a Selenium execution!

---

## 3. Building a WebDriver MCP Server

Imagine you want an AI to execute a specific Selenium test based on a Jira ticket it just read.

You would build an MCP Server in TypeScript that exposes a Tool:

```typescript
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import { exec } from "child_process";
import { z } from "zod";
// 1. Initialize the Server
const server = new McpServer({
  name: "Selenium-Automation-Server",
  version: "1.0.0"
});
// 2. Define a Tool that the AI can call
server.tool(
  "run_test",
  "Executes a specific Cucumber Tag via Selenium",
  {
    tag: z.string().describe("The cucumber tag to execute, e.g., @login")
  },
  async ({ tag }) => {
    return new Promise((resolve) => {
      console.log(`[MCP] AI requested execution of tag: ${tag}`);
      exec(`npm run test -- --tags "${tag}"`, (error, stdout) => {
        if (error) {
          resolve({
            content: [{ type: "text", text: `Test Failed:\n${stdout}` }]
          });
        } else {
          resolve({
            content: [{ type: "text", text: `Test Passed:\n${stdout}` }]
          });
        }
      });
    });
  }
);
// 3. Start the Server over Standard I/O
async function main() {
  const transport = new StdioServerTransport();
  await server.connect(transport);
  console.log("Selenium MCP Server running...");
}
main();
```

---

## 4. The Power of the Protocol

When an AI connects to this MCP Server, it will read the schema and realize it has the ability to run UI tests.

If you prompt an autonomous agent: *"Check Jira for open bugs. If there is a bug regarding the Login page, run the corresponding Selenium test to verify it."*

The AI will:
1. Use an Atlassian MCP Server to query Jira.
2. Find the bug report.
3. Use your custom Selenium MCP Server to call the `run_test` tool with the `@login` tag.
4. Read the `stdout` response from your framework.
5. Post a comment back on the Jira ticket with the results.

Zero human intervention.

## Conclusion

The Model Context Protocol transforms your Selenium framework from an isolated script into a powerful Tool that any AI model can leverage.

But setting up an MCP Server still requires the AI to be prompted by a human. What if the AI didn't need you to prompt it? What if the AI just ran continuously in the background, writing its own code, running its own tests, and fixing its own bugs?

Welcome to the ultimate endgame of software testing. In our next tutorial, we will explore **Autonomous Test Agents**.
