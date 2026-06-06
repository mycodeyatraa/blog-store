---
title: Mastering Postman: Introduction to Newman CLI
date: 09-Jan-2025
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [postman, api-testing, automation, newman, cli]
category: API Testing
categories: [Api Testing, Postman, Automation]
excerpt: >-
  Mastering Postman: Introduction to Newman CLI  > Key insight: A test suite that only runs when you manually click a button inside a GUI is a fragile test suite. To achieve true continuous integration,
readTime: 2 min read
---
> **Important Update:** To get the most out of this tutorial, we highly recommend running the official [MyCodeYatra Mock API Server](https://github.com/MYCodeYatra/myct-api-test-server) locally on `http://localhost:8080`. Replace any generic public API URLs in these examples with your local Mock Server endpoints!



# Mastering Postman: Introduction to Newman CLI

> **Key insight:** A test suite that only runs when you manually click a button inside a GUI is a fragile test suite. To achieve true continuous integration, your tests must run from the command line.

Up until now, we have been building our API tests entirely inside the Postman Desktop application. We've mastered dynamic variables, assertions, and OAuth automation. Our collection is robust and self-sufficient.

But in a modern DevOps environment, your tests need to run automatically every time a developer pushes code. The Postman UI cannot do this. You need a command-line interface. 

Enter **Newman**.

## 1. What is Newman?

Newman is Postman's open-source command-line Collection Runner. It allows you to run and test a Postman Collection directly from your terminal. Because it is a Node.js package, it can be executed on any machine—including Linux-based CI/CD servers like Jenkins, GitHub Actions, or GitLab CI.


![diagram_1](https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/mastering-postman-introduction-to-newman-cli/images/diagram_1.png)


## 2. Installation and Basic Usage

Because Newman is built on Node.js, you install it globally using `npm`:

```bash
npm install -g newman
```

To run a collection, you simply export your Postman Collection as a JSON file (e.g., `my_collection.json`) and run the `newman run` command:

```bash
newman run my_collection.json
```

Newman will read the file, sequentially execute every request, evaluate the test scripts, and print a beautifully formatted table of results directly into your terminal.

## 3. Passing Environments and Variables

Most real-world collections rely on Environment variables (like the `base_url` or `access_token` we configured in previous tutorials). 

If you run `newman run my_collection.json` without providing an environment, all your `{{base_url}}` variables will resolve as undefined and your tests will fail.

You must export your Postman Environment as a JSON file as well (e.g., `qa_env.json`) and pass it using the `-e` flag:

```bash
newman run my_collection.json -e qa_env.json
```

### Overriding Variables on the Fly

Sometimes you don't want to export a whole environment file, or you want to dynamically inject a secret token from your CI/CD pipeline. Newman allows you to override environment variables directly from the command line using the `--env-var` flag:

```bash
newman run my_collection.json -e qa_env.json --env-var "api_key=SECRET_XYZ123"
```

## Final Takeaways

Newman is the bridge between writing tests and automating them. By exporting your collections and environments to JSON, you can version control your test suite in Git alongside your application code. In our next tutorial, we will take Newman a step further and explore **Data-Driven Testing** by injecting massive CSV files directly into the CLI!
