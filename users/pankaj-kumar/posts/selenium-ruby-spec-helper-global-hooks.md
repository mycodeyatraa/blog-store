---
title: "Global Configuration with spec_helper.rb in RSpec"
date: "2025-03-26"
description: "Scale your test framework by extracting WebDriver lifecycle hooks into a global spec_helper.rb file."
tags: ["Selenium", "Ruby", "RSpec", "spec_helper", "Configuration"]
---

Welcome to Blog 14 of the **Selenium Ruby Mastery Series**! 

In Blog 13, we learned how to use RSpec `before(:each)` and `after(:each)` hooks to manage the WebDriver lifecycle. This was a massive improvement, but it still required us to paste those hooks at the top of every single `_spec.rb` file we created.

If you have 100 test files and decide you want to add an Implicit Wait to your driver, you still have to update 100 files! 

To achieve true centralized control, we use the **`spec_helper.rb`** file.

### What is `spec_helper.rb`?

In the RSpec testing framework, `spec_helper.rb` is a conventional configuration file. Instead of defining hooks inside a specific `RSpec.describe` block, you define them inside `RSpec.configure`. 

These global hooks will automatically wrap *every single test* in your entire suite!

### Step 1: Creating the Global Config

Let's extract our driver logic. Create a new file directly inside your `spec/` folder named `spec_helper.rb`:

```ruby
# spec/spec_helper.rb
require 'selenium-webdriver'
RSpec.configure do |config|
  # Global Before Hook: Runs before EVERY test in EVERY file
  config.before(:each) do
    options = Selenium::WebDriver::Chrome::Options.new
    options.add_argument('--headless')
    # By assigning @driver here, it is injected into the running spec!
    @driver = Selenium::WebDriver.for :chrome, options: options
    @driver.manage.timeouts.implicit_wait = 5
    puts "=> Global Setup: Browser Launched"
  end
  # Global After Hook: Runs after EVERY test
  config.after(:each) do
    if @driver
      @driver.quit
      puts "=> Global Teardown: Browser Terminated"
    end
  end
end
```

### Step 2: Writing ultra-clean Specs

Now, let's write a test script. We no longer need to `require 'selenium-webdriver'`, and we don't need any setup or teardown logic. We just `require_relative 'spec_helper'` and start writing business logic immediately using the globally injected `@driver`!

Create `spec/blog14_global_hooks_spec.rb`:

```ruby
require 'rspec'
require_relative 'spec_helper'
RSpec.describe 'Blog 14: Global Configuration' do
  # Look ma, no hooks! 
  it 'navigates to the Sandbox effortlessly' do
    @driver.navigate.to 'https://practice.mycodeyatra.com/'
    expect(@driver.title).to include('MyCodeYatra')
    puts "Test 1 Passed: The global @driver works!"
  end
  it 'navigates to the Login Page effortlessly' do
    @driver.navigate.to 'https://practice.mycodeyatra.com/#/login'
    expect(@driver.current_url).to include('login')
    puts "Test 2 Passed: The global @driver works again!"
  end
end
```

### Execution Output

When you run `rspec spec/blog14_global_hooks_spec.rb`:

```
Blog 14: Global Configuration
=> Global Setup: Browser Launched
Test 1 Passed: The global @driver works!
=> Global Teardown: Browser Terminated
=> Global Setup: Browser Launched
Test 2 Passed: The global @driver works again!
=> Global Teardown: Browser Terminated
Finished in 6.01 seconds
2 examples, 0 failures
```

### Conclusion

This is the turning point in your automation framework architecture. You now have a single, centralized file (`spec_helper.rb`) that controls the browser for your entire suite. If you want to switch from Chrome to Firefox tomorrow, you only have to change *one line of code* in the entire project!

Now that our framework is clean and scalable, it's time to tackle the hardest challenge in automation: **Data-Driven Testing**. 

In **Blog 15**, we will learn how to feed external data into our Ruby scripts by reading from CSV files!
