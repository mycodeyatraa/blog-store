---
title: "Headless Execution and ChromeOptions for CI/CD"
date: "2025-04-05"
description: "Prepare your Selenium framework for the cloud. Learn how to execute Ruby tests in headless mode and configure essential ChromeOptions for Jenkins and GitHub Actions."
tags: ["Selenium", "Ruby", "Headless", "ChromeOptions", "CI/CD"]
---

Welcome to Blog 24 of the **Selenium Ruby Mastery Series**! 

Up until now, you have been running your automation scripts on your personal laptop. You hit run, the Chrome browser physically opens on your screen, and you watch the magic happen.

But in the real world of Enterprise Engineering, tests are executed on **CI/CD Cloud Servers** (like Jenkins, AWS, or GitHub Actions). These servers run stripped-down Linux operating systems. They do not have computer monitors attached to them, and they do not have a Graphical User Interface (GUI) installed!

If your script attempts to open a physical Chrome window on a Linux server without a display, your test will crash instantly. 

The solution is **Headless Execution**.

### What is Headless Mode?

Headless mode tells the ChromeDriver to spin up the Chrome engine strictly in system memory, completely bypassing the GUI rendering process. The browser navigates, clicks, and evaluates JavaScript exactly like normal, but everything happens invisibly in the background.

*(Fun fact: Headless mode executes significantly faster because the computer doesn't waste CPU cycles drawing pixels on a screen!)*

### Configuring ChromeOptions for CI/CD

To survive in a cloud server (especially inside Docker containers), we need to pass a specific set of commands to Chrome.

Create `spec/blog24_headless_spec.rb`:

```ruby
require 'rspec'
require 'selenium-webdriver'
RSpec.describe 'Blog 24: Headless Execution and ChromeOptions' do
  it 'launches Chrome optimized for Linux CI/CD environments' do
    options = Selenium::WebDriver::Chrome::Options.new
    # 1. Enable Headless Mode
    options.add_argument('--headless')
    # 2. Bypass OS Security (Crucial for Docker containers and Jenkins)
    options.add_argument('--no-sandbox')
    # 3. Bypass /dev/shm memory limitations in Docker
    options.add_argument('--disable-dev-shm-usage')
    # 4. Disable hardware acceleration
    options.add_argument('--disable-gpu')
    # 5. The Golden Rule: Explicit Window Size!
    # Since there is no physical monitor, the headless browser defaults to a tiny 800x600 resolution.
    # This causes modern responsive websites to switch to "Mobile View", hiding buttons inside Hamburger Menus and breaking your tests!
    options.add_argument('--window-size=1920,1080')
    driver = Selenium::WebDriver.for :chrome, options: options
    driver.navigate.to 'https://practice.mycodeyatra.com/'
    # Let's prove the simulated window size is correct
    size = driver.manage.window.size
    puts "Browser launched silently in Headless Mode!"
    puts "Simulated Screen Resolution: #{size.width}x#{size.height}"
    expect(size.width).to eq(1920)
    expect(size.height).to eq(1080)
    driver.quit
  end
end
```

### Breaking Down the Configuration

*   `--no-sandbox`: Security sandboxes on Linux servers often clash with ChromeDriver's root permissions. This flag disables the sandbox.
*   `--disable-dev-shm-usage`: Docker containers have a very small shared memory partition (`/dev/shm`). Chrome is a memory hog and will crash the container if this isn't disabled.
*   `--window-size=1920,1080`: Without a physical monitor to tell Chrome how big to draw itself, it draws a tiny box. You *must* force it into a 1080p resolution so your WebElements don't overlap or hide behind responsive mobile breakpoints!

### Execution Output

```
Blog 24: Headless Execution and ChromeOptions
Browser launched silently in Headless Mode!
Simulated Screen Resolution: 1920x1080
  launches Chrome optimized for Linux CI/CD environments
Finished in 2.11 seconds
1 example, 0 failures
```

### Conclusion

Your Ruby framework is officially "Cloud Ready." If you take these exact options and place them into your global `spec_helper.rb`, you can safely push your code to any Jenkins server or GitHub Repository and execute it flawlessly via CI/CD pipelines!

In **Blog 25**, we will explore how to supercharge our test execution speed by running our RSpec suites concurrently using **Parallel Execution**!
