---
title: "Managing WebDriver Delays: Implicit vs. Explicit Waits"
date: "2025-03-17"
description: "Stop using sleep! Learn how to stabilize your Ruby test automation suite using Implicit and Explicit Waits with Selenium WebDriver."
tags: ["Selenium", "Ruby", "Waits", "Explicit Wait", "Synchronization"]
---

Welcome to Blog 5 of the **Selenium Ruby Mastery Series**! 

In our previous scripts, you might have noticed a command like `sleep 2`. Hardcoded sleeps halt your Ruby execution entirely. If the element appears in 0.5 seconds, you just wasted 1.5 seconds. If you have 100 tests with a `sleep 2`, you've artificially bloated your suite execution time by over 3 minutes! Worse, if the network is slow and the element takes 3 seconds, your test fails anyway.

To fix this, Selenium provides two smart synchronization strategies: **Implicit Waits** and **Explicit Waits**.

### 1. Implicit Waits (The Global Safety Net)

An Implicit Wait tells the WebDriver to poll the HTML DOM for a specified amount of time before throwing a `NoSuchElementError`. 

*   **Scope:** Global. Once set, it applies to *every* `find_element` call for the entire life of the driver session.
*   **Behavior:** If the element is found in 1 second, it proceeds immediately. If not, it keeps trying until the timeout is reached.

```ruby
driver.manage.timeouts.implicit_wait = 10
```

*Warning:* Implicit waits only check if an element *exists* in the DOM. They do not care if the element is actually visible or clickable!

### 2. Explicit Waits (The Precision Tool)

An Explicit Wait is applied to a specific, local condition. It pauses execution until a specific condition evaluates to `true`. This is crucial for modern web apps where an element might be present in the HTML (so Implicit wait passes) but has `display: none` or is covered by a loading spinner.

In Ruby, we use `Selenium::WebDriver::Wait.new`.

```ruby
wait = Selenium::WebDriver::Wait.new(timeout: 10)
element = wait.until { driver.find_element(id: 'my-btn').displayed? }
```

### Practical Example: Tackling Lazy Loading

Let's write a Ruby script to tackle the Lazy Loading page on our [Sandbox Application](https://practice.mycodeyatra.com).

Create `spec/blog5_waits_spec.rb`:

```ruby
require 'selenium-webdriver'
require 'rspec'
RSpec.describe 'Blog 5: Synchronization and Waits' do
  it 'demonstrates Implicit and Explicit Waits' do
    options = Selenium::WebDriver::Chrome::Options.new
    options.add_argument('--headless')
    driver = Selenium::WebDriver.for :chrome, options: options
    # 1. Global Implicit Wait
    driver.manage.timeouts.implicit_wait = 10
    driver.navigate.to 'https://practice.mycodeyatra.com/#/lazy-loading'
    puts "Navigated to Lazy Loading Page"
    # Implicit wait handles this. It polls until 'lazy-button' is added to the DOM.
    lazy_button = driver.find_element(id: 'lazy-button')
    lazy_button.click
    puts "Clicked the delayed button!"
    # 2. Explicit Wait
    # We must wait for the success message to not just exist, but be VISIBLE.
    wait = Selenium::WebDriver::Wait.new(timeout: 15)
    success_message = wait.until do
      element = driver.find_element(id: 'success-message')
      # The block loops until this returns a truthy value (the element itself)
      element if element.displayed?
    end
    puts "Visible message retrieved: #{success_message.text}"
    expect(success_message.text).to include('Success')
    driver.quit
  end
end
```

### Execution Output

```
Blog 5: Synchronization and Waits
Navigated to Lazy Loading Page
Clicked the delayed button!
Visible message retrieved: Success! You caught the lazy element.
  demonstrates Implicit and Explicit Waits
Finished in 6.45 seconds
1 example, 0 failures
```

### Conclusion

By using Implicit and Explicit Waits, your scripts are now dynamically responsive to network speeds. They will execute as fast as the application allows, and wait patiently when things are slow—completely eliminating the need for `sleep`!

While `wait.until { ... }` is great, it can get repetitive. In **Blog 6**, we will level up our Explicit Waits by diving into Custom Expected Conditions to make our Ruby code incredibly fluent and reusable!
