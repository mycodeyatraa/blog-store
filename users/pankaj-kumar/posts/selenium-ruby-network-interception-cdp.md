---
title: "Network Interception with Chrome DevTools Protocol"
date: "2025-04-11"
description: "Go beyond UI testing. Learn how to use the Chrome DevTools Protocol (CDP) in Ruby to intercept, mock, and manipulate HTTP network traffic."
tags: ["Selenium", "Ruby", "CDP", "Network Interception", "API Mocking"]
---

Welcome to Blog 30 of the **Selenium Ruby Mastery Series**! 

Historically, Selenium was strictly a "DOM interaction" tool. It could click buttons and type text, but it had no idea what was happening behind the scenes. If a button click triggered an API call, Selenium couldn't see the API request or its response.

With the release of Selenium 4, everything changed. Selenium 4 introduced direct integration with the **Chrome DevTools Protocol (CDP)**.

### What is CDP?

When you press `F12` in Chrome, you open the DevTools window (Console, Network tab, Application tab). CDP is the underlying API that powers those tools. By tapping into CDP, Selenium can now do things that used to be impossible: simulating slow 3G networks, going completely offline, or even rewriting HTTP API responses on the fly!

### Practical Example: Simulating Offline Mode

Have you ever wondered how your web application behaves if the user loses their Wi-Fi connection right before they click "Submit"? Let's write a test that dynamically turns off the browser's internet connection!

Create `spec/blog30_cdp_spec.rb`:

```ruby
require 'rspec'
require_relative 'spec_helper'
RSpec.describe 'Blog 30: Chrome DevTools Protocol (CDP)' do
  it 'simulates offline network conditions dynamically' do
    # 1. Enable the Network Domain
    # We must explicitly tell CDP that we want to control the Network
    puts "Enabling Chrome DevTools Network Domain..."
    @driver.execute_cdp('Network.enable')
    # 2. Emulate an Offline State!
    # We use execute_cdp and pass the exact parameters the Chrome API expects
    puts "Simulating OFFLINE network conditions..."
    @driver.execute_cdp('Network.emulateNetworkConditions', {
      offline: true,
      latency: 0,
      downloadThroughput: 0,
      uploadThroughput: 0
    })
    # 3. Attempt to Navigate
    # This will instantly throw a net::ERR_INTERNET_DISCONNECTED error!
    begin
      @driver.navigate.to 'https://practice.mycodeyatra.com/'
    rescue StandardError => e
      puts "=> Expected Error Caught! The browser has no internet connection."
      puts "=> Error Message: #{e.message}"
    end
    # 4. Restore the Network
    puts "Restoring ONLINE network conditions..."
    @driver.execute_cdp('Network.emulateNetworkConditions', {
      offline: false,
      latency: 0,
      downloadThroughput: -1, # -1 means unlimited
      uploadThroughput: -1
    })
    # 5. Navigate Again
    @driver.navigate.to 'https://practice.mycodeyatra.com/'
    puts "Successfully navigated to the site after restoring the network!"
    expect(@driver.title).to include('MyCodeYatra')
  end
end
```

### Breaking Down the Magic

*   **`execute_cdp(command, parameters)`**: This method is the bridge between Ruby and the Chromium engine. You pass the exact string command (like `'Network.emulateNetworkConditions'`) and a Hash of parameters.
*   **Offline Simulation**: By setting `offline: true`, we sever the browser's connection to the outside world *without* actually turning off the Wi-Fi on our physical computer. This means our test runner can still communicate with the ChromeDriver!

### Execution Output

```
Blog 30: Chrome DevTools Protocol (CDP)
=> Global Setup: Browser Launched
Enabling Chrome DevTools Network Domain...
Simulating OFFLINE network conditions...
=> Expected Error Caught! The browser has no internet connection.
=> Error Message: unknown error: net::ERR_INTERNET_DISCONNECTED
Restoring ONLINE network conditions...
Successfully navigated to the site after restoring the network!
=> Global Teardown: Browser Terminated
  simulates offline network conditions dynamically
Finished in 3.61 seconds
1 example, 0 failures
```

### Conclusion

By leveraging the Chrome DevTools Protocol, your Selenium framework is no longer restricted to the UI layer. You can simulate mobile bandwidth constraints, mock geolocation coordinates, and test offline progressive web app (PWA) behavior!

In **Blog 31**, we are going to look at the final piece of the Advanced Series puzzle: **Logging and Generating Custom HTML Reports**!
