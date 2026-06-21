---
title: "Mastering Explicit Waits (WebDriverWait) in Ruby"
date: "2025-04-02"
description: "Stop using blunt implicit waits and Thread.sleep. Learn how to use Selenium's explicit WebDriverWait to dynamically synchronize with complex web elements."
tags: ["Selenium", "Ruby", "Synchronization", "Explicit Waits", "WebDriverWait"]
---

Welcome to Blog 21 of the **Selenium Ruby Mastery Series**! 

Way back in Blog 5, we introduced `implicit_wait`. It was a great blunt instrument for beginners, telling the browser to wait globally for any element to exist in the HTML DOM. 

However, in modern React/Angular applications, an element might exist in the DOM (so `implicit_wait` passes instantly), but it might be completely invisible, covered by a loading spinner, or disabled. If you try to click it, your test will crash.

Today, we are learning the industry standard for synchronization: **Explicit Waits**.

### What is an Explicit Wait?

Instead of waiting globally for *presence*, an Explicit Wait allows you to pause your script and wait for a specific condition to be met on a specific element. (e.g., "Wait exactly for the Login Button to become clickable, up to 10 seconds, checking every 0.5 seconds.")

### Implementing Explicit Waits in Ruby

In Ruby, we utilize the `Selenium::WebDriver::Wait` class. Let's write a script that interacts with the Waits Sandbox on our [Live Application](https://practice.mycodeyatra.com/#/waits).

Create `spec/blog21_explicit_waits_spec.rb`:

```ruby
require 'rspec'
require_relative 'spec_helper'
RSpec.describe 'Blog 21: Explicit Waits' do
  it 'uses explicit synchronization to handle delayed elements' do
    @driver.navigate.to 'https://practice.mycodeyatra.com/#/waits'
    # 1. Initialize the Wait object
    # Timeout = Max wait time. Interval = How often to check the condition.
    wait = Selenium::WebDriver::Wait.new(timeout: 10, interval: 0.5)
    puts "Clicking the 'Start Timer' button..."
    @driver.find_element(id: 'start-timer-btn').click
    puts "Waiting up to 10 seconds for the delayed element to appear..."
    # 2. The wait.until loop
    # Selenium will execute this block every 0.5 seconds.
    # It will ONLY break the loop and continue your script if the block returns true (or an object).
    delayed_element = wait.until do
      element = @driver.find_element(id: 'delayed-text')
      # Return the element ONLY if it is actually visible on the screen!
      element if element.displayed?
    end
    # 3. Execute Actions
    # We are 100% guaranteed the element is visible at this point.
    puts "Element appeared! Text: #{delayed_element.text}"
    expect(delayed_element.text).to include('I am here')
    # 4. Negative Waits (Waiting for disappearance)
    puts "Waiting for the element to disappear..."
    wait.until do
      begin
        !@driver.find_element(id: 'delayed-text').displayed?
      rescue Selenium::WebDriver::Error::NoSuchElementError
        true # The element was destroyed in the DOM, so return true!
      end
    end
    puts "The element has successfully vanished!"
  end
end
```

### Breaking Down the Code

*   **`Wait.new(timeout: 10)`**: If 10 seconds pass and the condition is still not met, Selenium will throw a `TimeoutError` and fail the test. This prevents your tests from hanging infinitely!
*   **`wait.until do ... end`**: This is a polling loop. If the code inside the block throws a `NoSuchElementError` or evaluates to `false`/`nil`, Selenium simply swallows the error, waits `interval` seconds, and tries again!
*   **Negative Waits**: Waiting for an element to *disappear* (like a loading spinner finishing) is just as important as waiting for an element to appear. We handle this by rescuing the `NoSuchElementError` and returning `true`.

### Execution Output

```
Blog 21: Explicit Waits
=> Global Setup: Browser Launched
Clicking the 'Start Timer' button...
Waiting up to 10 seconds for the delayed element to appear...
Element appeared! Text: I am here
Waiting for the element to disappear...
The element has successfully vanished!
=> Global Teardown: Browser Terminated
  uses explicit synchronization to handle delayed elements
Finished in 8.32 seconds
1 example, 0 failures
```

### Conclusion

By transitioning from Implicit Waits to Explicit Waits, you are eliminating "flaky" tests from your framework entirely. Your scripts will now dynamically adjust their speed to match the exact rendering time of your web application!

In **Blog 22**, we will learn how to handle one of the trickiest modern web components: **File Uploads and Downloads in Selenium**!
