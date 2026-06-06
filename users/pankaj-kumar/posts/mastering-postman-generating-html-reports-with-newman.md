---
title: Mastering Postman: Generating HTML Reports with Newman
date: 2025-01-11
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [postman, api-testing, automation, newman, reporting]
category: API Testing
categories: [Api Testing, Postman, Automation]
excerpt: >-
  Mastering Postman: Generating HTML Reports with Newman  > Key insight: A test suite that runs silently in the background is useless if stakeholders can't easily read the results. You need beautiful, s
readTime: 2 min read
---

# Mastering Postman: Generating HTML Reports with Newman

> **Key insight:** A test suite that runs silently in the background is useless if stakeholders can't easily read the results. You need beautiful, shareable reports.

In our last tutorial, we learned how to run data-driven Postman collections from the command line using Newman. While the terminal output looks cool to a developer, your QA manager and Product Owner probably don't want to read thousands of lines of command-line logs.

They want a dashboard. They want pie charts. They want to know exactly which request failed and why.

Enter **newman-reporter-htmlextra**.

## 1. What is newman-reporter-htmlextra?

Newman supports custom reporters. The default reporter simply prints to the console (`cli`), but the open-source community created `htmlextra`, a reporter that generates a stunning, interactive HTML dashboard out of your test results.


![diagram_1](https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/mastering-postman-generating-html-reports-with-newman/images/diagram_1.png)


## 2. Installing the Reporter

Since it is a separate NPM package, you need to install it globally alongside Newman:

```bash
npm install -g newman-reporter-htmlextra
```

## 3. Generating Your First Report

To generate the report, you use the same `newman run` command we learned earlier, but you add the `-r` (reporter) flag. We will tell Newman to use *both* the CLI reporter (so we can still watch it run) and the `htmlextra` reporter.

```bash
newman run my_collection.json -e dev_env.json -r cli,htmlextra
```

When the test finishes, you won't see anything new in the terminal. However, if you look in your current directory, Newman will have automatically created a new folder called `newman/`. Inside that folder will be an HTML file (e.g., `my_collection-2025-01-11.html`).

## 4. Exploring the Dashboard

When you open the HTML file in your browser, you will see a massive upgrade over the terminal output:

1. **Summary Tab:** Displays the total run duration, total requests, assertions passed/failed, and a beautiful donut chart of your success rate.
2. **Total Requests Tab:** A breakdown of every single API call. If a call failed, you can expand it to see the exact Request Headers, Request Body, Response Headers, and the specific assertion that threw the error.
3. **Failed Tests Tab:** A filtered view showing *only* the tests that broke, allowing you to debug issues instantly without scrolling through hundreds of passed tests.

## Final Takeaways

By combining Postman, Newman, and `htmlextra`, you have built a complete, professional-grade test reporting engine. You can now zip this HTML file and attach it to an email or a Slack message. However, manually running these tests and generating reports on your laptop isn't true CI/CD. In our next tutorial, we will take this exact command and automate it inside a **GitHub Actions Pipeline**!
