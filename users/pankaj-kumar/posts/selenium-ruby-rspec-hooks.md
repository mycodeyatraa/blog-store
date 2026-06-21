---
title: "Mastering RSpec Hooks for WebDriver Lifecycle Management"
date: "2025-03-25"
description: "Stop duplicating WebDriver initialization! Learn how to use RSpec before(:each) and after(:each) hooks to cleanly manage your Selenium browser sessions."
tags: ["Selenium", "Ruby", "RSpec", "Hooks", "Lifecycle"]
---

Welcome to Blog 13 of the **Selenium Ruby Mastery Series**! 

Up until now, every single one of our RSpec `it` blocks has started and ended exactly the same way:

```ruby
it 'does something' do
  # 1. Boilerplate Setup
  options = Selenium::WebDriver::Chrome::Options.new
  driver = Selenium::WebDriver.for :chrome, options: options
  # 2. Actual Test Code
  driver.navigate.to '...'
  # 3. Boilerplate Teardown
  driver.quit
end
```

If you have 100 tests, you are writing that setup and teardown code 100 times. Not only does this violate the **DRY** (Don't Repeat Yourself) principle, but it's also incredibly dangerous. If an assertion fails on step 2, Ruby throws an Exception and the `it` block instantly terminates. The `driver.quit` line is *never reached*, leaving a ghost Chrome process running permanently in your computer's memory!

To solve this, we use **RSpec Hooks**.

### What are RSpec Hooks?

Hooks allow you to define blocks of code that RSpec will automatically execute at specific points in the test lifecycle.

1. **`before(:each)`**: Executes automatically right *before* every `it` block.
2. **`after(:each)`**: Executes automatically right *after* every `it` block—**guaranteed to run even if the test fails!**

### Practical Example: Centralizing Browser Management

Let's write a script containing multiple test cases against our [Live Sandbox](https://practice.mycodeyatra.com). We will extract the driver initialization and teardown into our hooks.

Create `spec/blog13_rspec_hooks_spec.rb`:

```ruby
require 'selenium-webdriver'
require 'rspec'
RSpec.describe 'Blog 13: RSpec Hooks for Lifecycle Management' do
  # Runs BEFORE every test
  before(:each) do
    options = Selenium::WebDriver::Chrome::Options.new
    options.add_argument('--headless')
    # Notice we use @driver (Instance Variable) so the `it` blocks can access it!
    @driver = Selenium::WebDriver.for :chrome, options: options
    @driver.manage.timeouts.implicit_wait = 5
    puts "=> before(:each) executed: Browser started."
  end
  # Runs AFTER every test (Even on Failure!)
  after(:each) do
    if @driver
      @driver.quit
      puts "=> after(:each) executed: Browser terminated."
    end
  end
  # Test 1
  it 'verifies the Sandbox Homepage title' do
    @driver.navigate.to 'https://practice.mycodeyatra.com/'
    expect(@driver.title).to include('MyCodeYatra')
    puts "Test 1 Passed: Homepage Title verified."
  end
  # Test 2
  it 'verifies the Login Page URL' do
    @driver.navigate.to 'https://practice.mycodeyatra.com/#/login'
    expect(@driver.current_url).to include('login')
    puts "Test 2 Passed: Login URL verified."
  end
end
```

### Breaking Down the Code

*   **Instance Variables (`@driver`)**: In Ruby, standard local variables (like `driver = ...`) defined inside `before(:each)` will disappear the moment the hook finishes executing. By adding the `@` symbol, we upgrade it to an instance variable, allowing it to persist and be shared with the `it` blocks and the `after(:each)` block!
*   **Guaranteed Teardown**: Because `after(:each)` runs regardless of test success or failure, your test environment is now memory-leak proof. Chrome will always be closed safely.

### Execution Output

```
Blog 13: RSpec Hooks for Lifecycle Management
=> before(:each) executed: Browser started.
Test 1 Passed: Homepage Title verified.
=> after(:each) executed: Browser terminated.
=> before(:each) executed: Browser started.
Test 2 Passed: Login URL verified.
=> after(:each) executed: Browser terminated.
Finished in 6.05 seconds
2 examples, 0 failures
```

### Conclusion

Your RSpec files are now incredibly clean! By centralizing your setup and teardown logic, your `it` blocks are free to focus exclusively on business logic and assertions.

However, if we have 50 different `_spec.rb` files, do we have to copy-paste these hooks into every single file? Absolutely not!

In **Blog 14**, we will introduce the `spec_helper.rb` file, teaching you how to configure global hooks that apply to your entire automation framework!
