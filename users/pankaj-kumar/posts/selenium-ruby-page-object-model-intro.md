---
title: "Introduction to the Page Object Model (POM) in Ruby"
date: "2025-03-23"
description: "Stop writing spaghetti code! Learn how to architect your test automation framework using the Page Object Model design pattern in Ruby."
tags: ["Selenium", "Ruby", "POM", "Page Object Model", "Framework Architecture"]
---

Welcome to Blog 11 of the **Selenium Ruby Mastery Series**! 

Congratulations, you have officially graduated from the Fundamentals and entered the **Intermediate Series**. 

Until now, we have been writing "Spaghetti Code." Our RSpec files contained driver initialization, hardcoded locators, data inputs, and assertions all mashed together in a single block. If a developer changes the ID of the username field tomorrow, you would have to manually find and update every single `spec` file where you typed `driver.find_element(id: 'username')`!

To fix this, the automation industry uses the **Page Object Model (POM)**.

### What is the Page Object Model?

POM is a design pattern that encourages the separation of **Business Logic** (the assertions in your test script) from **Technical Implementation** (the locators and driver clicks).

In POM, you create a dedicated Ruby Class for every physical page of your web application.

*   **Variables/Methods:** Represent the Locators (Web Elements) on that page.
*   **Action Methods:** Represent the interactions a user can perform (e.g., `login_as`, `search_for_item`).

### Step 1: Creating the Page Class

Let's build a `LoginPage` class for the [Live Sandbox Application](https://practice.mycodeyatra.com/#/login).

Create a new folder called `pages` and inside it, create `pages/login_page.rb`:

```ruby
class LoginPage
  # Constructor: We must pass the driver from the test to the page!
  def initialize(driver)
    @driver = driver
  end
  # --- Locators ---
  # Encapsulate locators into reusable methods
  def username_field
    @driver.find_element(id: 'username')
  end
  def password_field
    @driver.find_element(name: 'password')
  end
  def login_button
    @driver.find_element(xpath: "//button[@type='submit']")
  end
  # --- Actions ---
  # High-level business actions
  def navigate_to
    @driver.navigate.to('https://practice.mycodeyatra.com/#/login')
  end
  def login_as(username, password)
    username_field.send_keys(username)
    password_field.send_keys(password)
    login_button.click
  end
end
```

### Step 2: Refactoring the Test Script

Now, let's write our RSpec test. Notice how clean, readable, and devoid of raw `find_element` calls it is!

Create `spec/blog11_pom_spec.rb`:

```ruby
require 'selenium-webdriver'
require 'rspec'
require_relative '../pages/login_page'
RSpec.describe 'Blog 11: Page Object Model' do
  it 'performs a login using the LoginPage Object' do
    options = Selenium::WebDriver::Chrome::Options.new
    options.add_argument('--headless')
    driver = Selenium::WebDriver.for :chrome, options: options
    # 1. Instantiate the Page Object, passing the driver to it
    login_page = LoginPage.new(driver)
    # 2. Execute High-Level Actions
    login_page.navigate_to
    sleep 2 # Let React render
    login_page.login_as('admin', 'admin123')
    # 3. Assert
    puts "Successfully logged in using the Page Object Model!"
    expect(driver.title).to include('MyCodeYatra')
    driver.quit
  end
end
```

### Why is POM so Powerful?

1. **Reusability:** You can now write 50 different tests that require logging in, and you only write the `login_as` code once.
2. **Maintainability:** If the UI changes, you update the locator inside `login_page.rb` exactly *once*. All 50 tests instantly inherit the fix.
3. **Readability:** Your RSpec files now read like plain English. `login_page.login_as('admin', 'pass')` is infinitely easier for a non-technical manager to read than `driver.find_element(id: 'foo').send_keys`.

### Execution Output

```
Blog 11: Page Object Model
Successfully logged in using the Page Object Model!
  performs a login using the LoginPage Object
Finished in 4.30 seconds
1 example, 0 failures
```

### Conclusion

You have taken your first major step toward building an enterprise-grade automation framework. 

However, passing the `@driver` around and calling `.find_element` repeatedly every time a method runs can be slightly inefficient. In **Blog 12**, we will optimize our Page Objects by introducing the concept of **Lazy Evaluation and Caching** in Ruby!
