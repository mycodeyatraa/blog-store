---
title: "Controlling Execution with RSpec Tags and Filters"
date: "2025-03-30"
description: "Stop running your entire test suite for every minor change. Learn how to use RSpec metadata tags to isolate and execute specific subsets of your automated tests."
tags: ["Selenium", "Ruby", "RSpec", "Tags", "Test Execution"]
---

Welcome to Blog 18 of the **Selenium Ruby Mastery Series**! 

Imagine you have a test suite containing 5,000 automated UI tests. A developer just made a tiny change to the Login page and asks you, *"Can you run the tests to make sure I didn't break anything?"* 

If you type `rspec` in your terminal, RSpec will execute all 5,000 tests, which could take 10 hours! You only want to run the 50 tests related to the Login page, or perhaps your 20 ultra-critical "Smoke" tests.

To solve this, RSpec provides **Metadata Tags**.

### What are RSpec Tags?

Tags are simple symbols or key-value pairs that you append to your `describe` or `it` blocks. They act as labels. Once tests are labeled, you can instruct the RSpec command-line runner to *only* execute tests possessing a specific label.

### Step 1: Applying Tags to Tests

Let's create a spec file with various tags applied.

Create `spec/blog18_rspec_tags_spec.rb`:

```ruby
require 'rspec'
require_relative 'spec_helper'
# You can tag an entire group of tests!
RSpec.describe 'Blog 18: RSpec Tags', type: :ui do
  # A basic tag: 'smoke: true'
  it 'performs a critical application smoke test', smoke: true do
    @driver.navigate.to 'https://practice.mycodeyatra.com/'
    expect(@driver.title).to include('MyCodeYatra')
    puts "Executed Smoke Test!"
  end
  # A different tag: 'regression: true'
  it 'performs a deep functional regression test', regression: true do
    @driver.navigate.to 'https://practice.mycodeyatra.com/#/form'
    expect(@driver.current_url).to include('form')
    puts "Executed Regression Test!"
  end
  # A test can have MULTIPLE tags!
  it 'validates the login flow', smoke: true, login: true do
    @driver.navigate.to 'https://practice.mycodeyatra.com/#/login'
    expect(@driver.current_url).to include('login')
    puts "Executed Login Smoke Test!"
  end
end
```

### Step 2: Executing Tests by Tag

Now, let's look at the power of the command line.

#### Run ONLY Smoke Tests

To run only the tests tagged with `smoke`, use the `--tag` (or `-t`) flag:

`rspec spec/blog18_rspec_tags_spec.rb --tag smoke`

**Output:**
```
Blog 18: RSpec Tags
=> Global Setup: Browser Launched
Executed Smoke Test!
=> Global Teardown: Browser Terminated
=> Global Setup: Browser Launched
Executed Login Smoke Test!
=> Global Teardown: Browser Terminated
Finished in 6.22 seconds
2 examples, 0 failures
```
*(Notice that the Regression test was completely skipped!)*

#### Run ONLY Regression Tests

`rspec spec/blog18_rspec_tags_spec.rb --tag regression`

*(This will only run the 1 regression test).*

#### Run Everything EXCEPT Smoke Tests

Sometimes, you want to run your full suite *excluding* certain tests (like slow or flaky ones). You can prepend a `~` to the tag name to negate it!

`rspec spec/blog18_rspec_tags_spec.rb --tag ~smoke`

*(This will skip the 2 smoke tests and only run the regression test).*

### Conclusion

RSpec Tags give you ultimate, granular control over your test execution pipeline. When you eventually integrate your framework into a CI/CD pipeline (like Jenkins or GitHub Actions), you can configure it so that every code commit automatically triggers the `smoke` tag, while the nightly pipeline triggers the full `regression` tag.

In **Blog 19**, we are going to look at another critical aspect of Test Architecture: **Taking Automated Screenshots on Failure**!
