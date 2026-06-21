---
title: "Handling JavaScript Alerts, Prompts, and Confirmations"
date: "2025-03-21"
description: "Master the SwitchTo API in Ruby. Learn how to accept, dismiss, and extract text from un-inspectable JavaScript alert popups."
tags: ["Selenium", "Ruby", "Alerts", "SwitchTo", "JavaScript"]
---

Welcome to Blog 9 of the **Selenium Ruby Mastery Series**! 

Have you ever clicked a button on a website, and a small, native browser popup appeared asking "Are you sure you want to delete this?"? 

If you try to right-click and "Inspect Element" on that popup, you'll realize you can't! That's because it isn't made of HTML. It's a native JavaScript Alert rendered by the browser OS, meaning standard `find_element` commands will completely fail.

To handle these, Selenium provides the **`switch_to`** API.

### The Three Types of JavaScript Alerts

1. **Simple Alert**: Only displays a message and an "OK" button.
2. **Confirmation Alert**: Displays a message with "OK" and "Cancel" buttons.
3. **Prompt Alert**: Displays a message, an input text field, and "OK/Cancel" buttons.

### The `switch_to.alert` API

In Ruby, when an alert appears, we must shift WebDriver's focus away from the HTML page and onto the native alert using `driver.switch_to.alert`.

This returns an `Alert` object with four vital methods:
*   `.accept` (Clicks OK)
*   `.dismiss` (Clicks Cancel)
*   `.text` (Extracts the message text)
*   `.send_keys` (Types text into a Prompt)

### Practical Example: Mastering the Alerts Sandbox

Let's write an RSpec script that navigates to the Alerts page on our [Live Sandbox Application](https://practice.mycodeyatra.com/#/alerts) and conquers all three alert types!

Create `spec/blog9_alerts_spec.rb`:

```ruby
require 'selenium-webdriver'
require 'rspec'
RSpec.describe 'Blog 9: Handling JavaScript Alerts' do
  it 'switches to and handles Alerts, Confirmations, and Prompts' do
    options = Selenium::WebDriver::Chrome::Options.new
    options.add_argument('--headless')
    driver = Selenium::WebDriver.for :chrome, options: options
    # Navigate to the Alerts Practice Page
    driver.navigate.to 'https://practice.mycodeyatra.com/#/alerts'
    # 1. Simple Alert (OK only)
    driver.find_element(id: 'alert-btn').click
    simple_alert = driver.switch_to.alert
    puts "Simple Alert Text: #{simple_alert.text}"
    simple_alert.accept
    puts "Accepted the simple alert!"
    # 2. Confirmation Alert (OK and Cancel)
    driver.find_element(id: 'confirm-btn').click
    confirm_alert = driver.switch_to.alert
    puts "Confirmation Alert Text: #{confirm_alert.text}"
    confirm_alert.dismiss  # Clicking Cancel!
    puts "Dismissed the confirmation alert!"
    # 3. Prompt Alert (Input Field)
    driver.find_element(id: 'prompt-btn').click
    prompt_alert = driver.switch_to.alert
    puts "Prompt Alert Text: #{prompt_alert.text}"
    # Type into the alert before accepting it
    prompt_alert.send_keys('Selenium Ruby Mastery')
    prompt_alert.accept
    # Assert that the HTML page updated with our inputted text
    result_text = driver.find_element(id: 'prompt-result').text
    expect(result_text).to include('Selenium Ruby Mastery')
    puts "Successfully inputted text into the JS Prompt!"
    driver.quit
  end
end
```

### Execution Output

```
Blog 9: Handling JavaScript Alerts
Simple Alert Text: I am a simple alert
Accepted the simple alert!
Confirmation Alert Text: Press a button!
Dismissed the confirmation alert!
Prompt Alert Text: Please enter your name
Successfully inputted text into the JS Prompt!
  switches to and handles Alerts, Confirmations, and Prompts
Finished in 2.95 seconds
1 example, 0 failures
```

### Important Warning

If you call `driver.switch_to.alert` when there is no alert on the screen, Selenium will instantly throw a `NoAlertPresentError`. If your application has a slight delay before the alert appears, you should use an **Explicit Wait** (`Selenium::WebDriver::Wait`) to pause execution until the alert is present!

### Conclusion

You can now break out of the HTML DOM and interact directly with native browser dialogues. The `switch_to` API is incredibly versatile. 

In **Blog 10**, we will use this exact same API to solve one of the most infamously difficult automation challenges: Managing Multiple Browser Tabs, Windows, and embedded iFrames!
