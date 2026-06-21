---
title: "Keyboard and Mouse Actions using ActionBuilder"
date: "2025-03-20"
description: "Go beyond clicking. Learn how to simulate complex physical interactions like hover, double-click, and drag-and-drop using the ActionBuilder class in Ruby."
tags: ["Selenium", "Ruby", "ActionBuilder", "Mouse", "Keyboard"]
---

Welcome to Blog 8 of the **Selenium Ruby Mastery Series**! 

Standard WebDriver methods like `.click` and `.send_keys` are great for basic forms, but modern web applications are highly interactive. What if you need to hover over a navigation menu to reveal a dropdown? What if you need to double-click a row to edit it, or drag an item across the screen?

Standard methods cannot do this. To simulate raw, physical hardware interactions, we must use Selenium's **ActionBuilder** class.

### The Power of `driver.action`

In Ruby, the ActionBuilder is accessed via the `driver.action` method. This object allows us to chain multiple low-level physical interactions together into a single "sequence" and execute them all at once.

*Crucial Rule:* Action sequences do **not** execute until you call the `.perform` method at the end of the chain!

### Practical Example: The Actions Playground

Let's write a script that navigates to the Actions page on our [Live Sandbox Application](https://practice.mycodeyatra.com/#/actions) and performs four complex hardware maneuvers.

Create `spec/blog8_actionbuilder_spec.rb`:

```ruby
require 'selenium-webdriver'
require 'rspec'
RSpec.describe 'Blog 8: ActionBuilder Interactions' do
  it 'performs complex mouse and keyboard interactions' do
    options = Selenium::WebDriver::Chrome::Options.new
    options.add_argument('--headless')
    driver = Selenium::WebDriver.for :chrome, options: options
    # Navigate to the Actions Practice Page
    driver.navigate.to 'https://practice.mycodeyatra.com/#/actions'
    # 1. Hover (Mouse Move)
    hover_target = driver.find_element(id: 'hover-me')
    driver.action.move_to(hover_target).perform
    puts "Successfully hovered over the target!"
    # 2. Double Click
    double_click_btn = driver.find_element(id: 'double-click-btn')
    driver.action.double_click(double_click_btn).perform
    puts "Successfully double clicked the button!"
    # 3. Drag and Drop
    source = driver.find_element(id: 'draggable')
    target = driver.find_element(id: 'droppable')
    # This is a shortcut for click_and_hold -> move_to -> release
    driver.action.drag_and_drop(source, target).perform
    puts "Successfully dragged and dropped the element!"
    # 4. Complex Keyboard Sequences
    # We will click an input box, hold down SHIFT, type 'hello', and release SHIFT
    input_field = driver.find_element(id: 'keyboard-input')
    driver.action
          .click(input_field)
          .key_down(:shift)
          .send_keys('hello')
          .key_up(:shift)
          .perform
    puts "Typed 'HELLO' into the input field using SHIFT!"
    expect(input_field.attribute('value')).to eq('HELLO')
    driver.quit
  end
end
```

### Breaking Down the Code

*   **`move_to(element)`**: This simulates physically moving the mouse cursor to the center of the specified WebElement. This is required to trigger CSS `:hover` states or JavaScript `onmouseover` events!
*   **`drag_and_drop(source, target)`**: Internally, this executes three actions: it clicks and holds the source element, moves the mouse to the center of the target element, and releases the mouse click.
*   **`key_down(:shift)`**: Presses the SHIFT key and *holds it down*. Any subsequent `send_keys` commands will be capitalized, simulating a physical keyboard hold.
*   **`perform`**: The engine that compiles everything you chained together and ships it to the browser driver via the W3C Protocol.

### Execution Output

```
Blog 8: ActionBuilder Interactions
Successfully hovered over the target!
Successfully double clicked the button!
Successfully dragged and dropped the element!
Typed 'HELLO' into the input field using SHIFT!
  performs complex mouse and keyboard interactions
Finished in 4.78 seconds
1 example, 0 failures
```

### Conclusion

The `ActionBuilder` is a phenomenally powerful tool that gives you absolute control over the physical inputs of the browser. You can now build tests for complex data grids, drawing applications, or dynamic hover menus.

In **Blog 9**, we will explore another dimension of browser popups: handling JavaScript Alerts, Prompts, and Confirmations!
