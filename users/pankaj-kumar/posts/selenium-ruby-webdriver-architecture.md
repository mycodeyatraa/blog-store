---
title: "WebDriver Architecture & Your First Ruby Script"
date: "2025-03-14"
description: "Understand exactly how the W3C WebDriver Protocol communicates between Ruby and the browser, and write a detailed script to fetch live page metrics."
tags: ["Selenium", "Ruby", "Architecture", "W3C Protocol", "WebDriver"]
---

Welcome to Blog 2 of the **Selenium Ruby Mastery Series**! 

In the previous blog, we successfully set up our Ruby environment and ran our first headless test. Today, we are going to dive under the hood. Before you write hundreds of lines of automation code, you must understand **how** that code actually controls the browser. 

### The Selenium WebDriver Architecture

Selenium WebDriver is not a magic wand that directly clicks buttons inside a browser. It is a strictly architected system consisting of four major components:

1. **Selenium Language Bindings (Our Ruby Code):**
   When you write `driver.navigate.to`, the Ruby library creates an HTTP request containing this command.
   
2. **The W3C WebDriver Protocol:**
   This is the standard HTTP/JSON-based API protocol that Selenium uses. The Ruby client sends HTTP requests over a local server using this protocol. *(Note: Selenium 4 fully adopted the W3C protocol, replacing the older JSON Wire Protocol).*

3. **The Browser Driver:**
   This is a separate executable (like `chromedriver.exe` or `geckodriver`). It runs as a local HTTP server, listening for the W3C requests sent by your Ruby code. When it receives a request, it uses internal browser-specific APIs to execute the action.

4. **The Real Browser:**
   The actual Google Chrome or Mozilla Firefox window that physically executes the clicks, typing, and navigation. 

**Summary Flow:**
`Ruby Script` -> `HTTP Request` -> `Browser Driver Server` -> `Browser Execution` -> `HTTP Response` -> `Ruby Output`.

### Practical Example: Fetching Page Metrics

Let's write an RSpec specification that triggers multiple HTTP GET commands to extract basic page metrics from our [Sandbox Application](https://practice.mycodeyatra.com/#/sandbox).

Create a new file `spec/blog2_architecture_spec.rb`:

```ruby
require 'selenium-webdriver'
require 'rspec'
RSpec.describe 'Blog 2: WebDriver Architecture & Page Metrics' do
  it 'fetches and asserts page properties using the W3C WebDriver Protocol' do
    # 1. Start the WebDriver (Spins up ChromeDriver as an HTTP server)
    options = Selenium::WebDriver::Chrome::Options.new
    options.add_argument('--headless')
    driver = Selenium::WebDriver.for :chrome, options: options
    # 2. HTTP POST request: Navigate to the Sandbox UI
    driver.navigate.to 'https://practice.mycodeyatra.com/#/sandbox'
    # 3. HTTP GET request: Fetch Current URL
    actual_url = driver.current_url
    puts "Current URL: #{actual_url}"
    expect(actual_url).to include('practice.mycodeyatra.com')
    # 4. HTTP GET request: Fetch Page Title
    actual_title = driver.title
    puts "Page Title: #{actual_title}"
    expect(actual_title).to eq('MyCodeYatra | Test Automation Sandbox')
    # 5. HTTP GET request: Fetch Page Source Size
    source_size = driver.page_source.length
    puts "Page Source Size: #{source_size} bytes"
    expect(source_size).to be > 0
    # 6. HTTP DELETE request: Tear down the session
    driver.quit
  end
end
```

### Breaking Down the HTTP Requests

When you execute this script, here is what happens via the W3C protocol:

*   **`driver.navigate.to`**: Sends an HTTP `POST` to the driver instructing it to load a URL.
*   **`driver.current_url`**: Sends an HTTP `GET` to the driver asking for the URL. The driver queries the browser and returns a JSON payload with the URL string.
*   **`driver.title`**: Sends an HTTP `GET` to fetch the `<title>` tag of the page.
*   **`driver.page_source`**: Sends an HTTP `GET` to download the entire DOM (HTML).
*   **`driver.quit`**: Sends an HTTP `DELETE` to shut down the `chromedriver` server and close the browser memory processes. **Never forget to call `quit`**, or you will leave ghost chromedriver processes running in your machine's memory!

### Execution Output

When we run `rspec spec/blog2_architecture_spec.rb`, we receive the following output:

```
Blog 2: WebDriver Architecture & Page Metrics
Current URL: https://practice.mycodeyatra.com/#/sandbox
Page Title: MyCodeYatra | Test Automation Sandbox
Page Source Size: 5240 bytes
  fetches and asserts page properties using the W3C WebDriver Protocol
Finished in 2.91 seconds (files took 0.14 seconds to load)
1 example, 0 failures
```

### Conclusion

You now understand that Selenium is essentially a REST API client talking to a local server (`chromedriver`). Every command you write is translated into a network payload. 

In **Blog 3**, we will move beyond page metrics and learn how to actually locate and interact with physical Web Elements (buttons, text fields, and links) using Selenium's Locator Strategies!
