---
title: The Future is Here: AI Test Generation with LangChain
date: 01-Jul-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [python, selenium, ai, langchain, openai, test-generation]
category: AI & Future of Automation
categories: [AI & Future of Automation, Python, Automation]
excerpt: >-
  Is manual test automation dead? Learn how to use LangChain and Python to dynamically scrape application HTML and prompt OpenAI's GPT-4 to generate functional Pytest scripts in seconds!
readTime: 6 min read
---

# The Future is Here: AI Test Generation with LangChain

For the past 14 Phases, we have mastered the art of *manual* test automation. You learned how to inspect the DOM, write XPath selectors, configure Pytest fixtures, and build Docker Grids. 

But what if you didn't have to write the code at all? 

Welcome to **Phase 15: AI & The Future of Test Automation**. In this article, we will explore how Large Language Models (LLMs) are completely disrupting the QA industry. We will teach you how to use **LangChain** and Python to generate Pytest Selenium scripts entirely from natural language!

---

## 1. The Death of the Manual SDET?

Writing boilerplate Pytest code is tedious. When a developer creates a new Login page, an SDET traditionally spends an hour writing the Page Object Model, finding the locators, and writing the positive/negative assertions.

Today, LLMs like GPT-4 and Gemini can write that exact same code in 5 seconds. 

Does this mean SDETs are obsolete? **No.** It means the role of an SDET is evolving. Instead of *writing* tests, modern SDETs will *architect AI systems* that write the tests for them. 

---

## 2. Introducing LangChain

**LangChain** is a powerful Python framework designed to bridge the gap between your application code and LLMs. 

Instead of copying and pasting code from ChatGPT, we can use LangChain to programmatically query an LLM directly within our Python framework, feed it the HTML source code of our application, and ask it to output a functional Selenium test!

### Installation

```bash
pip install langchain langchain-openai
```

---

## 3. Building an AI Test Generator

Let's build a Python script that takes a URL, downloads the HTML, and asks OpenAI's GPT-4 to write a complete Pytest script to test the page!

**scripts/ai_test_generator.py**

```python
import os
import requests
from bs4 import BeautifulSoup
from langchain_openai import ChatOpenAI
from langchain.prompts import PromptTemplate
def generate_selenium_test(url, test_description):
    # 1. Fetch the raw HTML of the target page
    response = requests.get(url)
    html_content = response.text
    # 2. Clean the HTML (Remove heavy scripts/styles to save AI Tokens)
    soup = BeautifulSoup(html_content, 'html.parser')
    for script in soup(["script", "style"]):
        script.extract()
    clean_html = soup.body.prettify()[:5000] # Pass the first 5000 chars of the body
    # 3. Configure the LLM
    # Make sure OPENAI_API_KEY is set in your environment variables
    llm = ChatOpenAI(model_name="gpt-4", temperature=0.2)
    # 4. Define the AI Prompt Template
    prompt = PromptTemplate(
        input_variables=["html", "description"],
        template="""
        You are an expert SDET. I will provide you with the HTML of a webpage and a description of a test.
        Write a complete, functional Pytest Selenium script to perform this test.
        Rules:
        - Use standard Pytest fixtures (e.g., passing `driver`).
        - Use explicit waits (WebDriverWait).
        - Base your locators ONLY on the provided HTML.
        - Output ONLY valid Python code. Do not include markdown formatting or explanations.
        Test Description: {description}
        HTML Source:
        {html}
        """
    )
    # 5. Execute the AI Chain!
    print(f"[AI] Generating test for {url}...")
    formatted_prompt = prompt.format(html=clean_html, description=test_description)
    result = llm.invoke(formatted_prompt)
    # 6. Save the generated code to a file
    output_file = "tests/generated_test.py"
    with open(output_file, "w") as f:
        f.write(result.content)
    print(f"[AI] Test generated successfully: {output_file}")
# Example Usage:
generate_selenium_test(
    url="https://mycodeyatra.com/login",
    test_description="Verify that entering invalid credentials displays an error message."
)
```

### How does this work?
1. The script uses the `requests` library to fetch the DOM of our website.
2. We clean the HTML using `BeautifulSoup` to remove useless `<script>` tags, saving us money on API tokens.
3. We use LangChain's `PromptTemplate` to inject the clean HTML and our natural language requirement into a strict instruction set.
4. The LLM reads the HTML, identifies the `<input id="user">` and `<button id="submit">` tags, and writes a perfect, syntactically correct Pytest script using Explicit Waits!
5. The generated python file is instantly saved to our `tests/` directory, ready to be executed!

---

## 4. The Challenges of AI Generation

While this looks like magic, there are severe limitations:
- **Hallucinations:** The AI might invent locators that don't exist if the HTML is too complex.
- **Token Limits:** You cannot pass a massive Enterprise React SPA into an LLM context window yet.
- **Maintainability:** AI-generated code often lacks the strict architectural discipline of a human-designed Page Object Model.

## Conclusion

Generative AI is not replacing SDETs; it is giving them superpowers.
- **LangChain** allows you to seamlessly integrate LLMs into your Python automation workflows.
- By scraping the DOM and feeding it into an LLM via a strict `PromptTemplate`, you can auto-generate boilerplate test code in seconds.
- As token limits increase and models get cheaper, AI-assisted test generation will become the industry standard.

In our next article, we will tackle the most frustrating part of UI Automation—broken XPath locators—and show you how to build a **Self-Healing AI WebDriver!**
