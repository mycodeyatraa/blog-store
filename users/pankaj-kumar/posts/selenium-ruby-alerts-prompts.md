---
title: "Handling Browser Alerts, Prompts, and Confirmations"
date: "2025-04-10"
description: "Native JavaScript popups cannot be inspected or clicked normally. Learn how to accept, dismiss, and read text from browser alerts in Ruby."
tags: ["Selenium", "Ruby", "Alerts", "JavaScript Prompts"]
---

Welcome to Blog 29 of the **Selenium Ruby Mastery Series**! 

Have you ever filled out a form, clicked "Delete Profile", and a little grey box popped down from the top of the browser asking, *"Are you sure?"*

If you try to Right-Click > Inspect that box, nothing happens. It is not part of the HTML DOM! It is a native JavaScript dialog box rendered directly by Chrome. Because it is not HTML, standard Selenium `find_element` commands will completely fail to interact with it.

If an alert pops up and your script doesn't handle it, your test will freeze!

### The Alert API

To interact with native JavaScript popups, Selenium provides a dedicated `switch_to.alert` API.

There are three types of JavaScript Alerts:
1.  **Simple Alert:** Has a message and an "OK" button.
2.  **Confirmation:** Has a message, an "OK" button, and a "Cancel" button.
3.  **Prompt:** Has a message, a text input field, an "OK" button, and a "Cancel" button.

### Practical Example

Let's write a script that encounters all three types of alerts and handles them gracefully!

Create `spec/blog29_alerts_spec.rb`:

```ruby
require 'rspec'
require_relative 'spec_helper'
RSpec.describe 'Blog 29: JavaScript Alerts' do
  it 'handles simple alerts, confirmations, and prompts' do
    @driver.navigate.to 'https://practice.mycodeyatra.com/#/alerts'
    # ==========================================
    # 1. Simple Alert
    # ==========================================
    puts "Triggering a Simple Alert..."
    @driver.find_element(id: 'alert-btn').click
    # Pause Selenium's interaction with the HTML, and switch to the native popup
    alert = @driver.switch_to.alert
    puts "Alert says: #{alert.text}" # We can read the text!
    alert.accept # This simulates clicking 'OK'
    expect(@driver.find_element(id: 'alert-result').text).to include('You accepted the alert')
    # ==========================================
    # 2. Confirmation Alert
    # ==========================================
    puts "Triggering a Confirmation Alert..."
    @driver.find_element(id: 'confirm-btn').click
    alert = @driver.switch_to.alert
    puts "Alert says: #{alert.text}"
    alert.dismiss # This simulates clicking 'Cancel'
    expect(@driver.find_element(id: 'confirm-result').text).to include('You dismissed the confirmation')
    # ==========================================
    # 3. Prompt Alert
    # ==========================================
    puts "Triggering a Prompt Alert..."
    @driver.find_element(id: 'prompt-btn').click
    alert = @driver.switch_to.alert
    puts "Typing 'Pankaj Kumar' into the prompt..."
    alert.send_keys("Pankaj Kumar") # Type into the hidden input box
    alert.accept
    expect(@driver.find_element(id: 'prompt-result').text).to include('Pankaj Kumar')
    puts "All alerts handled successfully!"
  end
end
```

### Breaking Down the Code

*   `alert.text`: Extracts the string message inside the popup.
*   `alert.accept`: Clicks the positive action (OK/Yes).
*   `alert.dismiss`: Clicks the negative action (Cancel/No).
*   `alert.send_keys`: Injects text into the prompt's input field.

**Warning:** The moment you call `accept` or `dismiss`, the alert disappears and the `alert` object in Ruby is destroyed. You must extract `alert.text` *before* you accept/dismiss it!

### Execution Output

```
Blog 29: JavaScript Alerts
=> Global Setup: Browser Launched
Triggering a Simple Alert...
Alert says: I am a simple alert box!
Triggering a Confirmation Alert...
Alert says: Press a button!
Triggering a Prompt Alert...
Typing 'Pankaj Kumar' into the prompt...
All alerts handled successfully!
=> Global Teardown: Browser Terminated
  handles simple alerts, confirmations, and prompts
Finished in 2.92 seconds
1 example, 0 failures
```

### Conclusion

You no longer have to fear unexpected browser popups destroying your automated test runs. By utilizing `switch_to.alert`, you can smoothly navigate the native layers of the browser.

In **Blog 30**, we will learn how to mock API responses and manipulate HTTP Network traffic directly from Selenium by using **Chrome DevTools Protocol (CDP)**!
