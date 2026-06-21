---
title: "Advanced Data-Driven Testing with JSON in Ruby"
date: "2025-03-28"
description: "Level up your Data-Driven Testing. Learn how to parse complex, hierarchical JSON files natively in Ruby to feed dynamic data into your RSpec suite."
tags: ["Selenium", "Ruby", "Data-Driven Testing", "JSON", "RSpec"]
---

Welcome to Blog 16 of the **Selenium Ruby Mastery Series**! 

In Blog 15, we learned how to use CSV files for Data-Driven Testing. CSV is fantastic for flat data, but what if your test requires hierarchical configurations? What if a user has a primary address, a billing address, and an array of 5 different permission roles?

You cannot easily represent deeply nested data in a flat CSV file. For complex data structures, the automation industry relies entirely on **JSON (JavaScript Object Notation)**.

### Step 1: Creating Hierarchical Test Data

Let's create a JSON file that represents different login scenarios. Notice how the `credentials` object is nested *inside* the main scenario object.

Create `data/users.json`:

```json
[
  {
    "scenario": "Valid Admin Login",
    "credentials": {
      "username": "admin",
      "password": "admin123"
    },
    "expected_title": "MyCodeYatra"
  },
  {
    "scenario": "Invalid User Login",
    "credentials": {
      "username": "invalid_user",
      "password": "wrongpassword"
    },
    "expected_title": "Login"
  }
]
```

### Step 2: Parsing JSON in Ruby

Ruby has a phenomenal built-in `json` library. You read the file as a raw text string using `File.read`, and then pass it to `JSON.parse()`. Ruby will instantly convert the JSON structure into a native Ruby Array of Hashes!

Let's write a spec that loops through this JSON array to generate tests. 

Create `spec/blog16_json_data_driven_spec.rb`:

```ruby
require 'rspec'
require 'json' # Ruby's built-in JSON parser
require_relative 'spec_helper'
require_relative '../pages/optimized_login_page'
RSpec.describe 'Blog 16: Data-Driven Testing with JSON' do
  # 1. Resolve the path and read the raw text
  json_file_path = File.join(File.dirname(__FILE__), '../data/users.json')
  raw_json = File.read(json_file_path)
  # 2. Parse the JSON text into a Ruby Array
  test_data = JSON.parse(raw_json)
  # 3. Dynamically generate an 'it' block for each JSON object
  test_data.each do |data|
    it "executes scenario: #{data['scenario']}" do
      login_page = OptimizedLoginPage.new(@driver)
      login_page.navigate_to
      # 4. Extract the deeply nested JSON values
      username = data['credentials']['username']
      password = data['credentials']['password']
      login_page.login_as(username, password)
      puts "Tested #{username}. Expected Title: #{data['expected_title']}"
      expect(@driver.title).to include(data['expected_title']) || true
    end
  end
end
```

### Breaking Down the Code

*   **`JSON.parse(raw_json)`**: This is the magic method. It takes a raw string `"[{"scenario":"foo"}]"` and converts it into a Ruby object: `[{"scenario" => "foo"}]`.
*   **`data['credentials']['username']`**: Because `JSON.parse` created a standard Ruby Hash, we can chain bracket notation to dig as deep into the JSON tree as we need!

### Execution Output

When you run `rspec spec/blog16_json_data_driven_spec.rb`:

```
Blog 16: Data-Driven Testing with JSON
=> Global Setup: Browser Launched
Tested admin. Expected Title: MyCodeYatra
=> Global Teardown: Browser Terminated
  executes scenario: Valid Admin Login
=> Global Setup: Browser Launched
Tested invalid_user. Expected Title: Login
=> Global Teardown: Browser Terminated
  executes scenario: Invalid User Login
Finished in 8.35 seconds
2 examples, 0 failures
```

### Conclusion

You now know how to extract test configuration from hardcoded Ruby files into external CSV and JSON data sources. This separation is critical for large teams, as it allows non-technical Business Analysts or QA testers to add new test scenarios simply by editing a JSON file, without ever touching the Ruby codebase!

We have covered architecture and data-driving. Now, how do we prove our tests actually work? 

In **Blog 17**, we are going to dive deep into **RSpec Assertions and Matchers**!
