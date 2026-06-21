---
title: "Advanced Logging and Custom HTML Reports"
date: "2025-04-12"
description: "Stop reading messy terminal output. Learn how to generate beautiful, shareable HTML test execution reports for your management team using RSpec formatters."
tags: ["Selenium", "Ruby", "Reporting", "HTML", "RSpec"]
---

Welcome to Blog 31 of the **Selenium Ruby Mastery Series**! 

Up until this point, we have been running our automated tests from the terminal. If a test passes, we see a green dot. If it fails, we see a wall of red text. 

While terminal output is great for an engineer debugging locally, it is completely useless for a Project Manager or a QA Lead who wants to know: *"Did the nightly regression suite pass?"*

Management needs visual evidence. They need pie charts, summaries, and structured error logs. Today, we will learn how to extract beautiful HTML reports natively from RSpec!

### The Power of RSpec Formatters

You don't need to install any massive, complex third-party reporting libraries to generate good-looking reports in Ruby. RSpec actually ships with a built-in HTML Formatter!

All we have to do is change the command we type into the terminal.

### Step 1: Generating the Report

Instead of just typing `rspec`, we are going to append two new flags:
1.  `--format html`: Tells RSpec to output the results as HTML instead of plain text.
2.  `--out`: Tells RSpec exactly where to save that HTML file.

Let's run our entire `spec` directory and generate a report! Run this in your terminal:

```bash
rspec spec/ --format html --out reports/test_results.html
```

### Step 2: Understanding the Output

When you hit Enter, you will notice that your terminal stays completely silent. There are no green dots or red errors. This is because RSpec is routing 100% of the output stream directly into your new HTML file!

If you open your project directory, you will see a new folder named `reports` containing `test_results.html`. 

Double-click `test_results.html` to open it in Chrome.

You will see a beautifully structured document containing:
*   **A Header Summary**: Total execution time, Total Examples (Tests), and Total Failures.
*   **Collapsible Groups**: Every `describe` and `context` block is logically grouped.
*   **Detailed Stack Traces**: If a test failed, it will be highlighted in red. Clicking on it expands the exact error message and the line of code where it failed!

### Step 3: Combining Terminal Output AND HTML Output

If you run the command above, your terminal is silent. But what if you want to see the green dots in your terminal *while* it generates the HTML file in the background?

You can pass multiple formatters at the same time!

```bash
rspec spec/ --format documentation --format html --out reports/test_results.html
```

Now, the `documentation` formatter will print beautifully formatted text to your terminal screen as the tests run, while the `html` formatter silently builds the report in the background!

### Step 4: Automating Screenshots into the Report

In Blog 19, we configured our `spec_helper.rb` to automatically take a `.png` screenshot when a test fails. 

By using custom HTML reporting gems like `allure-rspec` or `rspec_html_reporter` in the future, you can actually configure RSpec to embed those failure screenshots *directly into the HTML file* next to the stack trace!

### Conclusion

You have now bridged the gap between Engineering and Management. By adding a simple command-line flag, you can generate shareable artifacts that prove your application is healthy and ready for deployment.

In **Blog 32**, we are going to dive into the world of **Database Testing**. We will learn how to bypass the UI entirely to verify that our form submissions actually wrote data to a SQL database!
