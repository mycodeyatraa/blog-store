---
title: "Automated Error Handling and Screenshots on Failure"
date: "2025-03-31"
description: "Never guess why a test failed again. Learn how to configure RSpec to automatically capture a screenshot of the browser exactly when an assertion fails."
tags: ["Selenium", "Ruby", "RSpec", "Screenshots", "Debugging"]
---

Welcome to Blog 19 of the **Selenium Ruby Mastery Series**! 

When you run a test suite overnight, and you wake up to find that 5 out of 100 tests failed, how do you know what went wrong? 

Did the webpage take too long to load? Was a popup blocking the button? Did the server return a 500 Error page? 

Without visual proof, debugging automated tests is a nightmare. Today, we are going to configure our global `spec_helper.rb` to automatically take a photograph of the screen the exact millisecond an RSpec assertion fails!

### Step 1: Upgrading our Global Configuration

In Blog 14, we created `spec_helper.rb`. We used an `after(:each)` hook to cleanly quit the driver. 

RSpec allows us to pass a special `example` variable into that hook. This variable contains the metadata of the test that just finished. If `example.exception` is present, it means the test failed!

Let's modify `spec/spec_helper.rb`:

```ruby
# spec/spec_helper.rb
require 'selenium-webdriver'
RSpec.configure do |config|
  config.before(:each) do
    options = Selenium::WebDriver::Chrome::Options.new
    options.add_argument('--headless')
    @driver = Selenium::WebDriver.for :chrome, options: options
    @driver.manage.timeouts.implicit_wait = 5
  end
  # We pass |example| into the hook!
  config.after(:each) do |example|
    if @driver
      # Check if the test threw an exception (failed)
      if example.exception
        # Create a screenshots folder if it doesn't exist
        Dir.mkdir('screenshots') unless Dir.exist?('screenshots')
        # Generate a unique filename using a Timestamp
        filename = "screenshots/error_#{Time.now.strftime('%Y%m%d_%H%M%S')}.png"
        # Capture the screenshot!
        @driver.save_screenshot(filename)
        puts "=> Test FAILED! Screenshot captured at: #{filename}"
      end
      @driver.quit
    end
  end
end
```

### Step 2: Forcing a Test to Fail

Let's write a test that is guaranteed to fail so we can watch our new screenshot engine in action.

Create `spec/blog19_screenshots_spec.rb`:

```ruby
require 'rspec'
require_relative 'spec_helper'
RSpec.describe 'Blog 19: Screenshots on Failure' do
  it 'purposely fails to demonstrate automated screenshots' do
    @driver.navigate.to 'https://practice.mycodeyatra.com/'
    # This assertion will FAIL because the title is actually "MyCodeYatra..."
    puts "Attempting to assert a bad title..."
    expect(@driver.title).to eq('This is the wrong title')
  end
end
```

### Execution Output

Run the failing test: `rspec spec/blog19_screenshots_spec.rb`

```
Blog 19: Screenshots on Failure
Attempting to assert a bad title...
=> Test FAILED! Screenshot captured at: screenshots/error_20250331_143022.png
F
Failures:
  1) Blog 19: Screenshots on Failure purposely fails to demonstrate automated screenshots
     Failure/Error: expect(@driver.title).to eq('This is the wrong title')
       expected: "This is the wrong title"
            got: "MyCodeYatra | Test Automation Sandbox"
Finished in 3.45 seconds
1 example, 1 failure
```

If you look in your project directory, you will now see a `screenshots` folder containing `error_20250331_143022.png`! You can open this image to see exactly what the browser looked like when RSpec triggered the failure.

### Conclusion

You have now built a self-documenting test suite. When tests fail in your CI/CD pipeline, you will immediately have visual evidence of the exact state of the UI, turning a 30-minute debugging session into a 30-second fix.

In **Blog 20**, we will explore one of the most powerful and dangerous concepts in JavaScript heavy applications: **Executing Custom JavaScript via WebDriver (`execute_script`)**!
