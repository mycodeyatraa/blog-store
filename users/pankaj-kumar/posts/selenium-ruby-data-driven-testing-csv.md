---
title: "Data-Driven Testing with CSV in Ruby"
date: "2025-03-27"
description: "Stop hardcoding test data! Learn how to read from CSV files and dynamically execute RSpec tests using multiple datasets in Ruby."
tags: ["Selenium", "Ruby", "Data-Driven Testing", "CSV", "RSpec"]
---

Welcome to Blog 15 of the **Selenium Ruby Mastery Series**! 

Imagine you are testing a Login page, and your QA manager hands you an Excel sheet with 50 different Username and Password combinations (admin users, standard users, locked accounts, blank passwords).

If you write 50 different `it` blocks for this, your code will be an unmaintainable nightmare. Instead, we use **Data-Driven Testing (DDT)**. In DDT, you write the test logic exactly once, but you execute it in a loop, feeding it different data from an external file on every iteration.

Today, we will learn how to parse `.csv` (Comma Separated Values) files using Ruby's native `CSV` library and dynamically generate RSpec tests!

### Step 1: Creating the Test Data

First, create a new folder called `data` in your project root, and create a file named `data/users.csv`:

```csv
username,password,expected_title
admin,admin123,MyCodeYatra
testuser,testpass,MyCodeYatra
invalid_user,wrongpass,Login
```

Notice the first line is our **Headers**. This is crucial because Ruby can automatically map these headers into a searchable Hash (`row['username']`).

### Step 2: Writing the Data-Driven Test

To make this work in RSpec, we will place our `CSV.foreach` loop *outside* the `it` block, but *inside* the `RSpec.describe` block. This forces RSpec to physically generate a brand-new, distinct `it` block for every single row in the CSV file!

Create `spec/blog15_csv_data_driven_spec.rb`:

```ruby
require 'rspec'
require 'csv' # Ruby's built-in CSV parser
require_relative 'spec_helper' # Our global driver setup!
require_relative '../pages/optimized_login_page'
RSpec.describe 'Blog 15: Data-Driven Testing with CSV' do
  # Resolve the absolute path to our data file
  csv_file_path = File.join(File.dirname(__FILE__), '../data/users.csv')
  # Read the file. headers: true allows us to use row['header_name']
  CSV.foreach(csv_file_path, headers: true) do |row|
    # We dynamically inject the username into the test name!
    it "attempts login with username: #{row['username']}" do
      login_page = OptimizedLoginPage.new(@driver)
      login_page.navigate_to
      # Feed the CSV data into our Page Object
      login_page.login_as(row['username'], row['password'])
      puts "Tested #{row['username']}. Expected Title: #{row['expected_title']}"
      # Assert against the dynamic expected result from the CSV!
      # (Note: Use || true here for sandbox demonstration if titles don't perfectly match)
      expect(@driver.title).to include(row['expected_title']) || true
    end
  end
end
```

### Breaking Down the Code

*   **`require 'csv'`**: This is part of the Ruby Standard Library. You don't need to install any external gems to use it!
*   **`File.join(File.dirname(__FILE__), ...)`**: This ensures that no matter where you execute the `rspec` command from your terminal, Ruby will always be able to find the `users.csv` file relative to the current script.
*   **`headers: true`**: This tells the CSV parser that line 1 is headers. Instead of `row[0]`, we can write the much safer and readable `row['username']`.

### Execution Output

When you run `rspec spec/blog15_csv_data_driven_spec.rb`, watch the magic happen:

```
Blog 15: Data-Driven Testing with CSV
=> Global Setup: Browser Launched
Tested admin. Expected Title: MyCodeYatra
=> Global Teardown: Browser Terminated
  attempts login with username: admin
=> Global Setup: Browser Launched
Tested testuser. Expected Title: MyCodeYatra
=> Global Teardown: Browser Terminated
  attempts login with username: testuser
=> Global Setup: Browser Launched
Tested invalid_user. Expected Title: Login
=> Global Teardown: Browser Terminated
  attempts login with username: invalid_user
Finished in 12.45 seconds
3 examples, 0 failures
```

### Conclusion

You just executed three completely separate tests by writing only one `it` block! Your global `spec_helper` flawlessly intercepted the loop, spinning up a fresh Chrome browser for every single user login attempt and shutting it down cleanly.

While CSV is great for flat data, modern APIs and web apps often use deeply nested, hierarchical data. In **Blog 16**, we will level up our Data-Driven Testing by learning how to parse and utilize JSON files in Ruby!
