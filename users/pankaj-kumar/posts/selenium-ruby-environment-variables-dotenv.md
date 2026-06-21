---
title: "Managing Secrets and Environment Variables"
date: "2025-04-07"
description: "Stop hardcoding passwords into your framework! Learn how to use the dotenv gem to securely manage environment-specific URLs, usernames, and passwords."
tags: ["Selenium", "Ruby", "Environment Variables", "Dotenv", "Security"]
---

Welcome to Blog 26 of the **Selenium Ruby Mastery Series**! 

Up until this point, whenever we needed to log into a website or navigate to a URL, we simply typed the string directly into our Ruby code.

```ruby
driver.navigate.to "https://qa-environment.com"
driver.find_element(id: 'pass').send_keys("SuperSecretPassword123!")
```

If you commit this code to GitHub, your company's proprietary QA environment and passwords are now exposed to everyone with access to the repository. Furthermore, what happens when you need to run your tests against the *Staging* environment instead of the *QA* environment? You would have to manually edit hundreds of Ruby files!

The solution is to separate **Configuration** from **Code** using Environment Variables.

### Step 1: Installing the `dotenv` Gem

The industry standard for managing environment variables in Ruby is the `dotenv` gem.

Open your `Gemfile` and add it:

```ruby
source 'https://rubygems.org'
gem 'selenium-webdriver'
gem 'rspec'
gem 'parallel_tests'
gem 'dotenv' # <--- Add this!
```

Run `bundle install` in your terminal.

### Step 2: Creating the `.env` File

At the root of your project directory, create a new file named exactly `.env`.

Inside this file, define your secrets:

```text
# .env
BASE_URL=https://practice.mycodeyatra.com
TEST_USER=admin
TEST_PASS=SuperSecretPassword123
```

**CRITICAL SECURITY WARNING:** You must IMMEDIATELY add `.env` to your `.gitignore` file. You should **NEVER** push your `.env` file to GitHub!

### Step 3: Accessing Secrets in Ruby

Now, let's create a test that logs into an application without exposing the password in the code!

Create `spec/blog26_dotenv_spec.rb`:

```ruby
require 'rspec'
require 'selenium-webdriver'
# This magical line reads the .env file and loads the variables into system memory!
require 'dotenv/load' 
RSpec.describe 'Blog 26: Environment Variables' do
  it 'securely logs in using hidden credentials' do
    # 1. We pull the Base URL from the ENV hash
    base_url = ENV['BASE_URL']
    options = Selenium::WebDriver::Chrome::Options.new
    options.add_argument('--headless')
    driver = Selenium::WebDriver.for :chrome, options: options
    # 2. We use String Interpolation to build the final URL!
    puts "Navigating to: #{base_url}/#/login"
    driver.navigate.to "#{base_url}/#/login"
    # 3. Pull our secret credentials
    username = ENV['TEST_USER']
    password = ENV['TEST_PASS']
    puts "Attempting login for user: #{username}"
    driver.find_element(id: 'username').send_keys(username)
    # We send the variable, NEVER the hardcoded string!
    driver.find_element(id: 'password').send_keys(password)
    driver.find_element(xpath: "//button[@type='submit']").click
    sleep 1
    # If the login succeeds, the URL changes
    expect(driver.current_url).not_to include('login')
    puts "Login successful without hardcoding secrets!"
    driver.quit
  end
end
```

### The CI/CD Pipeline Advantage

When you run this script locally, it pulls from your hidden `.env` file. 

But what happens when you push to Jenkins or GitHub Actions? Because you ignored the `.env` file, the CI/CD server won't have it! 

This is actually the **intended behavior**. In Jenkins or GitHub Actions, you securely configure the Environment Variables directly in their UI settings. When your Ruby script runs in the cloud, `ENV['BASE_URL']` seamlessly pulls from the server's securely stored pipeline secrets!

### Execution Output

```
Blog 26: Environment Variables
Navigating to: https://practice.mycodeyatra.com/#/login
Attempting login for user: admin
Login successful without hardcoding secrets!
  securely logs in using hidden credentials
Finished in 2.15 seconds
1 example, 0 failures
```

### Conclusion

Your framework is now Enterprise-Ready and secure. You can easily switch between QA, Staging, and Production environments simply by editing your local `.env` file, without touching a single line of Ruby code.

In **Blog 27**, we will tackle one of the most frustrating things in Selenium: **Managing Multiple Browser Windows and Tabs**!
