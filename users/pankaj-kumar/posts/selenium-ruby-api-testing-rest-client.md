---
title: "API Integration Testing with Rest-Client"
date: "2025-04-14"
description: "Speed up your test execution by seeding test data via REST APIs. Learn how to use the rest-client gem to perform HTTP GET and POST requests directly in Ruby."
tags: ["Selenium", "Ruby", "API Testing", "Rest-Client", "Data Seeding"]
---

Welcome to Blog 33 of the **Selenium Ruby Mastery Series**! 

Have you ever written a Selenium test to verify the "Delete User" functionality? In order to delete a user, you first have to *create* a user. 

If you use the UI to create the user, you spend 15 seconds navigating the registration form, clicking buttons, and waiting for page loads, *just* to get to the part of the test you actually care about (the Delete button).

What if we bypassed the UI and hit the backend API directly? We could create the user in **0.1 seconds**, and then instantly launch the browser to test the Delete button!

### The `rest-client` Gem

To make HTTP calls in Ruby, the industry standard gem is `rest-client`.

Open your `Gemfile` and add:
```ruby
gem 'rest-client'
```
*(Run `bundle install`)*

### Practical Example

We are going to use the public `reqres.in` API to demonstrate how to perform an HTTP GET request to read data, and an HTTP POST request to create data!

Create `spec/blog33_api_spec.rb`:

```ruby
require 'rspec'
require 'rest-client'
require 'json' # Built into Ruby, allows us to parse API responses
RSpec.describe 'Blog 33: API Integration Testing' do
  # ==========================================
  # 1. HTTP GET (Reading Data)
  # ==========================================
  it 'performs an HTTP GET request and parses the JSON response' do
    puts "Making an HTTP GET request to ReqRes API..."
    # Send the GET request
    response = RestClient.get('https://reqres.in/api/users/2')
    # Validate the Status Code
    puts "HTTP Status Code: #{response.code}"
    expect(response.code).to eq(200)
    # Convert the String response into a Ruby Hash
    body = JSON.parse(response.body)
    # Navigate the Hash to extract the data we want
    first_name = body['data']['first_name']
    last_name = body['data']['last_name']
    puts "Extracted User: #{first_name} #{last_name}"
    expect(first_name).to eq('Janet')
  end
  # ==========================================
  # 2. HTTP POST (Creating Data)
  # ==========================================
  it 'performs an HTTP POST request to seed data' do
    puts "\nMaking an HTTP POST request to create a user..."
    # Define our request body
    payload = {
      name: "Pankaj Kumar",
      job: "Automation Architect"
    }.to_json
    # Define our request headers
    headers = { content_type: :json, accept: :json }
    # Send the POST request
    response = RestClient.post('https://reqres.in/api/users', payload, headers)
    # Assert successful creation (201 Created)
    expect(response.code).to eq(201) 
    body = JSON.parse(response.body)
    puts "Successfully created User ID: #{body['id']} at #{body['createdAt']}"
  end
end
```

### Breaking Down the Code

1.  **`RestClient.get`**: Automatically hits the endpoint and blocks until the server responds.
2.  **`JSON.parse`**: Web APIs return a massive block of raw text. To navigate it in Ruby, we parse it into a Hash.
3.  **Data Seeding Strategy**: In a real framework, you would wrap that `RestClient.post` block into a helper method called `create_test_user_via_api()`. You would call this inside your `config.before(:each)` hook to instantly seed the database before Selenium even opens Chrome!

### Execution Output

```
Blog 33: API Integration Testing
Making an HTTP GET request to ReqRes API...
HTTP Status Code: 200
Extracted User: Janet Weaver
  performs an HTTP GET request and parses the JSON response
Making an HTTP POST request to create a user...
Successfully created User ID: 318 at 2024-04-14T10:05:00.123Z
  performs an HTTP POST request to seed data
Finished in 0.85 seconds
2 examples, 0 failures
```

### Conclusion

By combining UI interactions with Backend APIs, your framework becomes incredibly fast. Stop wasting time waiting for the UI to set up prerequisites; let the APIs do the heavy lifting in milliseconds!

In **Blog 34**, we will learn the final architectural pattern of this entire course: **The Page Object Model (POM)**!
