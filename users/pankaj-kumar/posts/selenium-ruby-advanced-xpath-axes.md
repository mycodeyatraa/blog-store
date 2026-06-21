---
title: "Advanced XPath Axes and Dynamic Elements in Ruby"
date: "2025-03-16"
description: "Conquer stubborn, dynamic web elements using Advanced XPath Axes like parent, following-sibling, and contains() in Selenium Ruby."
tags: ["Selenium", "Ruby", "XPath", "Advanced", "WebElements"]
---

Welcome to Blog 4 of the **Selenium Ruby Mastery Series**! 

In our last blog, we learned how to find web elements using static locators like IDs and Names. But what happens when you encounter a modern Single Page Application (SPA) built with React or Angular? Often, these frameworks generate dynamic IDs that change every time you refresh the page (e.g., `<input id="username_9f8d7">`).

If you hardcode `:id => "username_9f8d7"`, your test will fail tomorrow! To conquer dynamic UI elements, we must unleash the full power of **Advanced XPath**.

### What is XPath?

XPath (XML Path Language) is a syntax used to navigate through elements and attributes in an XML/HTML document. It allows us to locate elements not just by their static attributes, but by their relative relationships to other elements.

### Powerful XPath Functions

When attributes are partially dynamic, you can use these string-matching functions:

1. **`contains()`**: Finds an element whose attribute partially matches a string.
   *Example*: `//input[contains(@id, 'user')]` matches `user_123` and `username`.
2. **`starts-with()`**: Finds an element whose attribute begins with a string.
   *Example*: `//input[starts-with(@id, 'pass')]` matches `password_999`.

### XPath Axes (DOM Relationships)

XPath Axes allow you to locate an element based on its relationship to another element you can already easily find.

*   **`parent::`**: Selects the immediate parent of the current node.
*   **`ancestor::`**: Selects all parents (and grandparents) up to the root node.
*   **`following-sibling::`**: Selects all siblings that appear *after* the current node. (Incredibly useful for finding an `<input>` field sitting immediately after a specific `<label>`).
*   **`preceding-sibling::`**: Selects all siblings that appear *before* the current node.

### Practical Example: Navigating Dynamic Elements

Let's write a Ruby script that navigates the [Live Sandbox Application](https://practice.mycodeyatra.com/#/login) using nothing but advanced XPath relationships!

Create `spec/blog4_xpath_spec.rb`:

```ruby
require 'selenium-webdriver'
require 'rspec'
RSpec.describe 'Blog 4: Advanced XPath Axes' do
  it 'locates dynamic elements using XPath axes and functions' do
    options = Selenium::WebDriver::Chrome::Options.new
    options.add_argument('--headless')
    driver = Selenium::WebDriver.for :chrome, options: options
    # Navigate to the Login Page
    driver.navigate.to 'https://practice.mycodeyatra.com/#/login'
    sleep 2 # Simple wait for DOM rendering
    # 1. XPath contains() function
    # Even if the ID changes to 'user_999', 'user' is still inside it!
    username_field = driver.find_element(xpath: "//input[contains(@id, 'user')]")
    username_field.send_keys('admin')
    puts "Located username via XPath contains()"
    # 2. XPath starts-with() function
    password_field = driver.find_element(xpath: "//input[starts-with(@type, 'pass')]")
    password_field.send_keys('admin123')
    puts "Located password via XPath starts-with()"
    # 3. XPath Axes: ancestor
    # We find the form container, then search for the submit button inside it
    login_btn = driver.find_element(xpath: "//form[ancestor::div]//button[@type='submit']")
    login_btn.click
    puts "Located login button via XPath Ancestor relationship"
    # Assertion
    expect(driver.title).to include('MyCodeYatra')
    driver.quit
  end
end
```

### Execution Output

```
Blog 4: Advanced XPath Axes
Located username via XPath contains()
Located password via XPath starts-with()
Located login button via XPath Ancestor relationship
  locates dynamic elements using XPath axes and functions
Finished in 3.95 seconds
1 example, 0 failures
```

### Conclusion

You no longer need to rely on perfect, static IDs provided by developers! By mastering `contains()`, `starts-with()`, and DOM Axis relationships like `ancestor` and `following-sibling`, your Ruby automation scripts are now completely immune to dynamically shifting frontend framework IDs.

However, did you notice we used `sleep 2` in our script? Using hardcoded `sleep` statements is the **#1 cause of slow automation suites**. 

In **Blog 5**, we will eliminate `sleep` forever by teaching you how to manage WebDriver delays properly using Implicit and Explicit Waits!
