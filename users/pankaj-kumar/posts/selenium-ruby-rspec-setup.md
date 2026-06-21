---
title: "Introduction to Selenium with Ruby & RSpec (Setup & Installation)"
date: "2025-03-13"
description: "Kickstart your automation journey! Learn how to install Ruby, configure the Selenium WebDriver gem, and set up your first RSpec test project."
tags: ["Selenium", "Ruby", "RSpec", "Setup", "Automation"]
---

Welcome to the **Selenium Ruby Mastery Series**! 

Over the next 35 blogs, we will take you from a complete beginner to a highly skilled Automation Architect in Ruby. Ruby is famously loved for its beautiful, human-readable syntax. When combined with the **RSpec** testing framework, your automation code reads almost like plain English.

In this first blog, we will configure our environment, install the necessary dependencies (gems), and execute our very first Selenium test against our [Live Sandbox Application](https://practice.mycodeyatra.com/#/sandbox).

### Step 1: Installing Ruby

If you are on Windows, download the [RubyInstaller for Windows](https://rubyinstaller.org/). For macOS or Linux, use `rbenv` or `rvm`. 

Once installed, verify it by opening your terminal and typing:
```bash
ruby -v
```
You should see output similar to `ruby 3.2.2 (2023-03-30 revision e51014f4c1) [x64-mingw-ucrt]`.

### Step 2: Initializing the Project

Create a new directory for your automation project and navigate into it:

```bash
mkdir mcyt-sel-ruby
cd mcyt-sel-ruby
```

Ruby uses **Bundler** to manage project dependencies. Initialize a new `Gemfile`:

```bash
bundle init
```

### Step 3: Configuring the Gemfile

Open the generated `Gemfile` in your favorite editor (like VS Code or RubyMine) and add the Selenium and RSpec dependencies:

```ruby
source "https://rubygems.org"
gem "selenium-webdriver", "~> 4.18.0"
gem "rspec", "~> 3.13.0"
```

Save the file and install the gems by running:
```bash
bundle install
```

### Step 4: Writing Your First RSpec Test

RSpec expects test files to be placed in a folder named `spec` and end with `_spec.rb`. Let's create `spec/blog1_setup_spec.rb`:

```ruby
require 'selenium-webdriver'
require 'rspec'
RSpec.describe 'Blog 1: Environment Setup' do
  it 'successfully launches Google Chrome and navigates to the Practice Site' do
    # 1. Configure Chrome to run in Headless mode
    options = Selenium::WebDriver::Chrome::Options.new
    options.add_argument('--headless')
    # 2. Initialize the WebDriver
    driver = Selenium::WebDriver.for :chrome, options: options
    # 3. Navigate to the Sandbox UI
    driver.navigate.to 'https://practice.mycodeyatra.com/#/sandbox'
    # 4. Assert the Title matches expected value
    expect(driver.title).to eq('MyCodeYatra | Test Automation Sandbox')
    # 5. Output success message
    puts "Successfully navigated to: #{driver.title}"
    # 6. Tear down the browser session
    driver.quit
  end
end
```

### Understanding the Code

1. **`require 'selenium-webdriver'`**: Imports the Selenium 4 library into our Ruby script.
2. **`RSpec.describe` & `it`**: This is RSpec's elegant DSL (Domain Specific Language). It creates a highly readable, nested structure describing exactly what the test does.
3. **`Options.new`**: We configure Chrome to run in headless mode (without opening a visible window), which is faster and perfect for CI environments.
4. **`expect(...).to eq(...)`**: This is RSpec's powerful assertion engine. It verifies that the actual browser title equals our expected string.

### Step 5: Executing the Test

To run the test, simply execute RSpec from your terminal:

```bash
rspec spec/blog1_setup_spec.rb
```

### Execution Output

```
Blog 1: Environment Setup
Successfully navigated to: MyCodeYatra | Test Automation Sandbox
  successfully launches Google Chrome and navigates to the Practice Site
Finished in 2.34 seconds (files took 0.12 seconds to load)
1 example, 0 failures
```

### Conclusion

Congratulations! You have successfully configured your Ruby environment, installed Selenium 4 and RSpec via Bundler, and executed your first assertion against a live web page.

In **Blog 2**, we will dive deep into Selenium's architectural components to understand exactly how Ruby commands are translated into browser actions under the hood!
