---
title: "Slashing Execution Time: Parallel Test Execution"
date: "2025-04-06"
description: "Stop waiting hours for your test suite to finish. Learn how to configure the parallel_tests gem to run multiple RSpec tests simultaneously across CPU cores."
tags: ["Selenium", "Ruby", "Parallel Execution", "RSpec", "Optimization"]
---

Welcome to Blog 25 of the **Selenium Ruby Mastery Series**! 

Imagine you have a test suite containing 100 UI tests. If each test takes an average of 15 seconds to execute (opening the browser, navigating, clicking, asserting, and closing), your entire suite will take **25 minutes** to finish. 

If a developer makes a small code change and has to wait 25 minutes for the pipeline to pass, they will get frustrated. What if we could divide those 100 tests into 4 separate chunks and run them all at the exact same time? The execution time drops from 25 minutes to roughly **6 minutes**!

Today, we will learn how to achieve this using the `parallel_tests` gem!

### Step 1: Installing `parallel_tests`

By default, RSpec runs tests **sequentially** (one after another). To run them concurrently, we need an external library.

Open your `Gemfile` and add:

```ruby
source 'https://rubygems.org'
gem 'selenium-webdriver'
gem 'rspec'
gem 'parallel_tests'
```

Open your terminal and run `bundle install`.

### Step 2: Preparing Your Tests for Parallelism

Before you can run tests in parallel, you must ensure your tests are **Independent**. 

*   **Bad Architecture**: Test A creates a user, and Test B attempts to log in with the user Test A just created. If Test B runs at the exact same time as Test A, the user won't exist yet, and Test B will fail!
*   **Good Architecture**: Every single test handles its own complete setup and teardown. Tests do not rely on each other.

Because we have been using the `config.before(:each)` and `config.after(:each)` hooks in our `spec_helper.rb` to launch and destroy a fresh browser for *every* test, our framework is already perfectly designed for parallelism!

### Step 3: Executing in Parallel

Normally, you run your tests using:
`rspec spec/`

With the `parallel_tests` gem installed, you now have access to a new command-line executable: `parallel_rspec`.

To run your tests across 4 separate CPU cores simultaneously, type:

```bash
bundle exec parallel_rspec -n 4 spec/
```

### Breaking Down the Command

*   **`bundle exec`**: This ensures you are running the exact version of the gem installed in your project.
*   **`parallel_rspec`**: The custom runner that intercepts your specs and divides them up.
*   **`-n 4`**: Tells the runner to spawn 4 independent Ruby processes. If you have 20 specs, Process 1 will take the first 5, Process 2 will take the next 5, and so on. They will all launch their own Chrome browser at the same time!

### Execution Output

```
4 processes for 4 specs, ~ 1 specs per process
...
Blog 18: RSpec Tags
=> Global Setup: Browser Launched
Blog 19: Screenshots on Failure
=> Global Setup: Browser Launched
Blog 20: Executing Custom JavaScript
=> Global Setup: Browser Launched
Blog 21: Explicit Waits
=> Global Setup: Browser Launched
...
=> Global Teardown: Browser Terminated
=> Global Teardown: Browser Terminated
=> Global Teardown: Browser Terminated
=> Global Teardown: Browser Terminated
4 examples, 0 failures
Took 8.4 seconds
```

*(Notice how 4 browsers launched at the exact same time!)*

### Conclusion

Parallel execution is the secret to scaling enterprise automation frameworks. By combining **Headless Mode** (Blog 24) with **Parallel Execution** (Blog 25), you can easily run 500+ tests in a matter of minutes on a CI/CD server, providing instant feedback to your development team!

In **Blog 26**, we will look at how to structure our test data securely by **Managing Environment Variables and Secrets using dotenv**!
