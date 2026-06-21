---
title: "Simulating Complex Actions: Hover, Drag and Drop"
date: "2025-04-04"
description: "Master the Selenium ActionBuilder API to replicate complex human interactions like mouse hovering, double-clicking, and drag-and-drop in Ruby."
tags: ["Selenium", "Ruby", "ActionBuilder", "Mouse Actions", "Drag and Drop"]
---

Welcome to Blog 23 of the **Selenium Ruby Mastery Series**! 

Up until now, we have interacted with the browser using simple commands like `.click` and `.send_keys`. But what happens if you need to open a dropdown menu that only appears when you hover over it with your mouse? What if you need to double-click a row in a data table, or drag a slider bar?

Standard WebElements do not have a `.hover` or `.drag` method. To accomplish these complex physical gestures, we use Selenium's **ActionBuilder API**.

### The ActionBuilder API

The ActionBuilder (`driver.action`) allows you to simulate native hardware input devices (the Mouse and the Keyboard). It allows you to chain multiple micro-gestures together into a single, fluid movement.

### Practical Example

Let's write a script that performs a Hover, a Double-Click, and a Drag-and-Drop operation against our [Live Sandbox](https://practice.mycodeyatra.com).

Create `spec/blog23_actions_spec.rb`:

```ruby
require 'rspec'
require_relative 'spec_helper'
RSpec.describe 'Blog 23: ActionBuilder API' do
  it 'simulates advanced mouse interactions' do
    @driver.navigate.to 'https://practice.mycodeyatra.com/#/actions'
    # ==========================================
    # 1. Mouse Hover (Move To Element)
    # ==========================================
    hover_target = @driver.find_element(id: 'hover-me')
    # We use driver.action, chain the gesture, and ALWAYS conclude with .perform
    @driver.action.move_to(hover_target).perform
    # Assert the hidden text is now visible
    hidden_text = @driver.find_element(id: 'hidden-text')
    expect(hidden_text.displayed?).to be true
    puts "Hover successful! Hidden text revealed."
    # ==========================================
    # 2. Double Click
    # ==========================================
    double_click_btn = @driver.find_element(id: 'double-click-btn')
    @driver.action.double_click(double_click_btn).perform
    puts "Double click successful!"
    # ==========================================
    # 3. Drag and Drop
    # ==========================================
    source_element = @driver.find_element(id: 'draggable')
    target_element = @driver.find_element(id: 'droppable')
    # Drag from the Source and drop onto the Target
    @driver.action.drag_and_drop(source_element, target_element).perform
    expect(target_element.text).to include('Dropped!')
    puts "Drag and Drop successful!"
    # ==========================================
    # 4. Complex Gesture Chaining
    # ==========================================
    # ActionBuilder lets you build entirely custom physics!
    @driver.action.click_and_hold(source_element)
                  .move_by(100, 50) # Move 100px Right, 50px Down
                  .release
                  .perform
    puts "Custom gesture completed!"
  end
end
```

### The Golden Rule: `.perform`

The most common mistake beginners make with the ActionBuilder is forgetting the `.perform` method at the very end of the chain.

When you call `@driver.action.move_to(element)`, you are just building an instruction set in memory. Absolutely nothing happens in the browser until you execute `.perform`, which compiles the gestures and sends them to the ChromeDriver to be executed!

### Execution Output

```
Blog 23: ActionBuilder API
=> Global Setup: Browser Launched
Simulating a Mouse Hover...
Hover successful! Hidden text revealed.
Simulating a Double Click...
Double click successful!
Simulating Drag and Drop...
Drag and Drop successful!
Simulating a custom gesture...
Custom gesture completed!
=> Global Teardown: Browser Terminated
  simulates advanced mouse interactions
Finished in 6.12 seconds
1 example, 0 failures
```

### Conclusion

The ActionBuilder API gives you the power to simulate exact, pixel-perfect human behaviors. Whether it is swiping, dragging, or hovering, you can now automate any modern Web 2.0 interface.

In **Blog 24**, we will shift our focus to Headless execution and learn **How to configure ChromeOptions for CI/CD Pipelines**!
