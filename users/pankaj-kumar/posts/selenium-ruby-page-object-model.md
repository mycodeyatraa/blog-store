---
title: "The Page Object Model (POM): Architecting Your Framework"
date: "2025-04-15"
description: "Stop duplicating locators across hundreds of files! Learn how to implement the Page Object Model (POM) to create a clean, maintainable, and scalable test architecture."
tags: ["Selenium", "Ruby", "Page Object Model", "POM", "Architecture"]
---

Welcome to Blog 34 of the **Selenium Ruby Mastery Series**! 

Up until this point, we have been writing all of our locators (`id: 'username'`) and actions (`click`, `send_keys`) directly inside our RSpec `it` blocks.

If you have 50 different tests that all need to log in to the application, you have hardcoded `id: 'username'` 50 separate times. 

What happens if the developers change the UI, and the ID becomes `id: 'login-user'`? You now have to manually find and update 50 broken tests. This is a maintenance nightmare!

The solution to this problem is the **Page Object Model (POM)**.

### What is the Page Object Model?

POM is an industry-standard design pattern where you create a dedicated Ruby Class for every single Web Page in your application.

*   **The Page Class**: Contains all the Locators and Action Methods for that specific page. It *does not* contain Assertions!
*   **The Spec File**: Instantiates the Page Class, calls its methods, and performs the Assertions.

### Step 1: Creating the Page Class

Create a new directory in your project called `pages`. Inside it, create `login_page.rb`:

```ruby
# pages/login_page.rb
class LoginPage
  # 1. Capture the driver from the Spec file
  def initialize(driver)
    @driver = driver
  end
  # 2. Define Locators as Private Methods
  private
  def username_input
    @driver.find_element(id: 'username')
  end
  def password_input
    @driver.find_element(id: 'password')
  end
  def login_button
    @driver.find_element(xpath: "//button[@type='submit']")
  end
  # 3. Define Public Actions
  public
  def navigate_to
    @driver.navigate.to 'https://practice.mycodeyatra.com/#/login'
  end
  def perform_login(username, password)
    username_input.send_keys(username)
    password_input.send_keys(password)
    login_button.click
  end
end
```

By storing `username_input` here, if the ID ever changes, we only have to update it in **one single file**, and all 50 tests will instantly be fixed!

### Step 2: Using the Page in the Spec

Now, let's look at how incredibly clean our RSpec file becomes!

Create `spec/blog34_pom_spec.rb`:

```ruby
require 'rspec'
require_relative 'spec_helper'
require_relative '../pages/login_page'
RSpec.describe 'Blog 34: Page Object Model' do
  it 'logs in using the POM Architecture' do
    # 1. Instantiate the Page Object, passing our @driver
    login_page = LoginPage.new(@driver)
    # 2. Read the script like plain English!
    puts "Navigating to Login Page..."
    login_page.navigate_to
    puts "Performing Login as admin..."
    login_page.perform_login("admin", "password123")
    # 3. The Spec file handles the Assertions
    expect(@driver.current_url).not_to include('login')
    puts "Successfully logged in using POM!"
  end
end
```

### Why POM is Mandatory for Seniors

Notice how there is **zero Selenium Code** (`find_element`, `click`) inside the `blog34_pom_spec.rb` file! 

The test script reads like a plain English business requirement: "Navigate to page, Perform Login, Expect URL to change." This completely separates the *Business Logic* (the Spec) from the *Technical Implementation* (the Page Class).

### Execution Output

```
Blog 34: Page Object Model
=> Global Setup: Browser Launched
Navigating to Login Page...
Performing Login as admin...
Successfully logged in using POM!
=> Global Teardown: Browser Terminated
  logs in using the POM Architecture
Finished in 2.22 seconds
1 example, 0 failures
```

### Conclusion

You have officially graduated from writing basic automation scripts to designing Enterprise Framework Architectures. The Page Object Model is the foundation of every professional Selenium project in the world.

In our **35th and Final Blog**, we will wrap up this incredible journey by teaching you how to integrate your entire masterpiece into a **Jenkins CI/CD Pipeline**!
