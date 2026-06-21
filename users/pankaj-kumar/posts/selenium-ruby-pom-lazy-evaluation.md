---
title: "Optimizing Page Objects with Lazy Evaluation and Caching in Ruby"
date: "2025-03-24"
description: "Make your Page Objects blazingly fast. Learn how to implement Ruby's ||= operator for lazy evaluation and WebElement caching."
tags: ["Selenium", "Ruby", "POM", "Lazy Evaluation", "Caching"]
---

Welcome to Blog 12 of the **Selenium Ruby Mastery Series**! 

In Blog 11, we built a beautiful Page Object Model (POM) to separate our locators from our RSpec logic. However, our initial implementation had a hidden performance flaw.

Look at this method from our previous blog:
```ruby
def username_field
  @driver.find_element(id: 'username')
end
```
Every single time you call `login_page.username_field`, Ruby sends a brand-new HTTP request over the W3C protocol to the ChromeDriver, asking it to scan the entire HTML DOM to find the element. If you type into the field, clear it, and type again, you just triggered 3 identical DOM searches!

We can optimize this using **Ruby's Lazy Evaluation (`||=`)**.

### What is Lazy Evaluation?

In Ruby, the `||=` (or-equals) operator is a magical shortcut. 

```ruby
@my_variable ||= "Hello"
```
This translates to: *"If `@my_variable` is currently nil or false, assign 'Hello' to it. If it already has a value, do absolutely nothing and just return the existing value."*

### Step 1: Caching WebElements

We can use `||=` to **cache** our WebElements. The first time the method is called, it searches the DOM. Every subsequent time, it instantly returns the cached element from memory!

Create `pages/optimized_login_page.rb`:

```ruby
class OptimizedLoginPage
  def initialize(driver)
    @driver = driver
  end
  # The FIRST time this is called, it executes find_element.
  # The SECOND time, it instantly returns the cached @username_field.
  def username_field
    @username_field ||= @driver.find_element(id: 'username')
  end
  def password_field
    @password_field ||= @driver.find_element(name: 'password')
  end
  def login_button
    @login_button ||= @driver.find_element(xpath: "//button[@type='submit']")
  end
  def navigate_to
    @driver.navigate.to('https://practice.mycodeyatra.com/#/login')
  end
end
```

### Step 2: Putting it to the Test

Let's write a script that interacts with the `username_field` multiple times.

Create `spec/blog12_lazy_pom_spec.rb`:

```ruby
require 'selenium-webdriver'
require 'rspec'
require_relative '../pages/optimized_login_page'
RSpec.describe 'Blog 12: Lazy Evaluation in POM' do
  it 'caches WebElements to improve execution speed' do
    options = Selenium::WebDriver::Chrome::Options.new
    options.add_argument('--headless')
    driver = Selenium::WebDriver.for :chrome, options: options
    login_page = OptimizedLoginPage.new(driver)
    login_page.navigate_to
    sleep 2
    # 1st call: Scans the DOM via HTTP request
    login_page.username_field.send_keys('admin')
    # 2nd call: INSTANT. Uses memory cache.
    login_page.username_field.clear
    # 3rd call: INSTANT. Uses memory cache.
    login_page.username_field.send_keys('testuser')
    puts "Successfully utilized cached WebElements without re-querying the DOM!"
    expect(login_page.username_field.attribute('value')).to eq('testuser')
    driver.quit
  end
end
```

### The Danger of Caching: StaleElementReferenceException

Caching is incredibly fast, but it comes with a danger. If the web page refreshes, or if React/Angular completely destroys and re-renders the DOM node, your cached `@username_field` will point to a "Ghost" element that no longer exists on the screen. 

If you try to click it, Selenium will throw a `StaleElementReferenceException`.

**The Fix:** If you know the page will refresh, you must clear your cache! You can add a simple method to your Page Object:
```ruby
def clear_cache!
  @username_field = nil
end
```
Setting it to `nil` forces the `||=` operator to query the DOM again the next time it is called!

### Execution Output

```
Blog 12: Lazy Evaluation in POM
Successfully utilized cached WebElements without re-querying the DOM!
  caches WebElements to improve execution speed
Finished in 3.42 seconds
1 example, 0 failures
```

### Conclusion

By leveraging Ruby's native `||=` operator, you have eliminated redundant HTTP requests to the browser driver, making your automation suite significantly faster. 

However, you might have noticed we are still instantiating our `driver` directly inside every single `it` block. In **Blog 13**, we will introduce **RSpec Hooks (`before(:each)` and `after(:each)`)** to automatically manage our WebDriver lifecycle!
