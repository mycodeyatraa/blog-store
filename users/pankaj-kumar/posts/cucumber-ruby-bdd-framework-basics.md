---
title: Behavior-Driven Development: Cucumber-Ruby Basics & RSpec
date: 29-Feb-2026
lastUpdated: 29-Feb-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [selenium, ruby, cucumber, rspec, bdd, gherkin]
category: Selenium Ruby
categories: [Selenium Ruby, BDD, Cucumber]
excerpt: >-
  Harness Cucumber in its native Ruby environment! Build modular BDD feature files and RSpec step definitions with Selenium Ruby.
readTime: 8 min read
---

# Behavior-Driven Development: Cucumber-Ruby Basics & RSpec

Cucumber was originally created for the Ruby ecosystem. Combining **Cucumber-Ruby** with **Selenium WebDriver** and **RSpec** assertions provides the most expressive, elegant BDD framework available in automation.

---

## 1. Core Architecture of Cucumber-Ruby

```
 +--------------------------------+
 | Feature File (Gherkin Syntax)  |
 +--------------------------------+
                 |
                 v
 +--------------------------------+
 | Step Definitions (Ruby / RSpec)|
 +--------------------------------+
                 |
                 v
 +--------------------------------+
 | Selenium WebDriver Execution   |
 +--------------------------------+
```

---

## 2. Writing a Feature File

**features/login.feature**

```gherkin
Feature: Customer Portal Login
  As a registered user
  I want to log in with valid credentials
  So that I can access my personalized dashboard
 
  @smoke
  Scenario: Successful Login with Valid Credentials
    Given I navigate to the login page
    When I submit valid credentials "pankaj@mycodeyatra.com" and "Secret123"
    Then I should see the dashboard heading "Welcome, Pankaj"
```

---

## 3. Implementing Step Definitions in Ruby

**features/step_definitions/login_steps.rb**

```ruby
Given('I navigate to the login page') do
  @driver.navigate.to 'https://mycodeyatra.com/login'
end
 
When('I submit valid credentials {string} and {string}') do |username, password|
  @driver.find_element(id: 'username').send_keys(username)
  @driver.find_element(id: 'password').send_keys(password)
  @driver.find_element(id: 'login-btn').click
end
 
Then('I should see the dashboard heading {string}') do |expected_heading|
  heading_text = @driver.find_element(tag_name: 'h1').text
  expect(heading_text).to include(expected_heading)
end
```

---

## 4. Hooks and Environment Setup

**features/support/env.rb**

```ruby
require 'selenium-webdriver'
require 'rspec/expectations'
 
Before do
  options = Selenium::WebDriver::Chrome::Options.new
  options.add_argument('--headless')
  @driver = Selenium::WebDriver.for :chrome, options: options
  @driver.manage.timeouts.implicit_wait = 10
end
 
After do
  @driver.quit if @driver
end
```
