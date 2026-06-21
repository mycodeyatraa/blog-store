---
title: "Advanced Synchronization: Custom Waits in Ruby"
date: "2025-03-18"
description: "Master advanced UI synchronization. Learn to write your own custom polling conditions to wait for exact application states using Selenium::WebDriver::Wait."
tags: ["Selenium", "Ruby", "Waits", "Custom Conditions", "Synchronization"]
---

Welcome to Blog 6 of the **Selenium Ruby Mastery Series**! 

In Blog 5, we learned how to use Explicit Waits to pause execution until an element becomes visible. But what if "visible" isn't enough? 

Consider a scenario where an element is visible, but its text is rapidly updating from `"Status: Processing"` to `"Status: Complete"`. If you just wait for it to be visible, your test will grab the `"Processing"` text and immediately fail your assertion!

To solve this, we must build **Custom Wait Conditions**.

### How `Wait.until` Actually Works in Ruby

In Ruby, `Selenium::WebDriver::Wait#until` accepts a block. It will repeatedly execute the code inside the block every 0.5 seconds (by default). 

*   If the block evaluates to `false` or `nil`, it sleeps and tries again.
*   If the block evaluates to a **truthy** value (like a boolean `true`, or an Element object), the wait successfully terminates and returns that truthy value to you!

### Practical Example: Waiting for Regex Text Matches

Let's write a custom Ruby method that accepts a Regex pattern and forces the WebDriver to poll the DOM until the target element's text specifically matches that pattern. We will run this against our [Sandbox Application](https://practice.mycodeyatra.com).

Create `spec/blog6_custom_waits_spec.rb`:

```ruby
require 'selenium-webdriver'
require 'rspec'
RSpec.describe 'Blog 6: Custom Waits' do
  # 1. Our Custom Synchronization Helper Method
  def wait_for_text_to_match(driver, locator, regex, timeout = 10)
    wait = Selenium::WebDriver::Wait.new(
      timeout: timeout, 
      message: "Timeout waiting for text to match #{regex}"
    )
    wait.until do
      element = driver.find_element(locator)
      # If the text matches the Regex, return the element (Truthy -> stops polling)
      # Otherwise, return false (Falsy -> continues polling)
      element.text.match?(regex) ? element : false
    end
  end
  it 'uses custom conditions to wait for a specific UI state' do
    options = Selenium::WebDriver::Chrome::Options.new
    options.add_argument('--headless')
    driver = Selenium::WebDriver.for :chrome, options: options
    # 2. Navigate and Trigger Action
    driver.navigate.to 'https://practice.mycodeyatra.com/#/lazy-loading'
    driver.find_element(id: 'lazy-button').click
    # 3. Call our Custom Wait
    puts "Waiting for text to strictly match our Regex..."
    success_element = wait_for_text_to_match(driver, { id: 'success-message' }, /Success!/)
    puts "Custom wait succeeded! Found: #{success_element.text}"
    expect(success_element.text).to include('Success!')
    driver.quit
  end
end
```

### Breaking Down the Code

*   **`wait = Selenium::WebDriver::Wait.new(message: "...")`**: Passing a custom message is critical. If the wait times out after 10 seconds, Selenium will throw a `TimeoutError` containing this exact message, making debugging incredibly easy!
*   **`element.text.match?(regex) ? element : false`**: This is a beautiful Ruby ternary operator. If the text matches, it yields the `element` back to the test script so we can assert against it. If not, it yields `false`, signaling the `Wait` class to try again in 500 milliseconds.

### Execution Output

```
Blog 6: Custom Waits
Waiting for text to strictly match our Regex...
Custom wait succeeded! Found: Success! You caught the lazy element.
  uses custom conditions to wait for a specific UI state
Finished in 5.30 seconds
1 example, 0 failures
```

### Conclusion

You now have the tools to synchronize your automation suite against *any* imaginable UI state. Whether you need to wait for a color to change, an animation to complete, or a specific string of text to render, you can encapsulate that logic directly inside a custom Wait block!

In **Blog 7**, we will transition from basic clicks and typing to handling complex Web Elements—specifically Checkboxes, Radio Buttons, and native Dropdowns using Selenium's Select class!
