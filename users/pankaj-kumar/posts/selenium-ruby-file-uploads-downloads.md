---
title: "Handling File Uploads and Downloads in Selenium"
date: "2025-04-03"
description: "Learn how to bypass the native OS file picker window to upload files seamlessly, and configure Chrome to automatically download files without prompting."
tags: ["Selenium", "Ruby", "File Upload", "File Download", "Chrome Options"]
---

Welcome to Blog 22 of the **Selenium Ruby Mastery Series**! 

One of the most common questions beginners ask is: *"How do I make Selenium click the Browse button, open the Windows File Explorer, and select a file?"*

The answer is: **You don't.**

Selenium is a *web* automation tool. It has zero capability to interact with native Windows or Mac desktop applications (like the File Picker dialog box). If you click a "Browse" button and that OS window opens, your Selenium script will freeze forever, unable to control it.

Today, we will learn how to bypass the OS entirely!

### 1. Bypassing the File Picker for Uploads

When you look at the HTML of a file upload button, it is almost always an `<input type="file">` tag. 

Instead of clicking the button, we can simply locate the `input` element and use `.send_keys()` to send the **absolute path** of the file we want to upload! The browser handles the rest automatically.

### 2. Bypassing the "Save As" Prompt for Downloads

When you click a download link, Chrome often opens a native OS "Save As..." dialog box. Just like the upload picker, Selenium cannot click "Save" on this native window.

To fix this, we must pass **Custom Preferences** to the ChromeDriver *before* it launches, telling it to automatically download files to a specific folder without ever prompting the user.

### Practical Example

Let's write a script that configures our download directory and successfully uploads the `users.csv` file we created back in Blog 15!

Create `spec/blog22_file_upload_spec.rb`:

```ruby
require 'rspec'
require 'selenium-webdriver'
require 'fileutils' # For creating our download directory
RSpec.describe 'Blog 22: File Uploads and Downloads' do
  it 'uploads a file without opening the OS window' do
    options = Selenium::WebDriver::Chrome::Options.new
    options.add_argument('--headless')
    # ==========================================
    # 1. Configure Automatic Downloads
    # ==========================================
    # Create a custom folder in our project root
    download_dir = File.join(Dir.pwd, 'downloads')
    FileUtils.mkdir_p(download_dir) 
    # Tell Chrome to auto-download to this folder!
    options.add_preference('download.default_directory', download_dir)
    options.add_preference('download.prompt_for_download', false)
    # Note: We instantiate a fresh driver here to apply these custom options
    driver = Selenium::WebDriver.for :chrome, options: options
    # ==========================================
    # 2. File Uploading
    # ==========================================
    driver.navigate.to 'https://practice.mycodeyatra.com/#/sandbox'
    # Step A: Locate the hidden <input type="file"> element
    file_input = driver.find_element(id: 'upload-file')
    # Step B: Get the ABSOLUTE path to the file. 
    # Relative paths (like 'data/users.csv') will FAIL!
    file_to_upload = File.absolute_path(File.join(File.dirname(__FILE__), '../data/users.csv'))
    # Step C: Send the path directly into the input tag!
    puts "Uploading file: #{file_to_upload}"
    file_input.send_keys(file_to_upload)
    puts "File uploaded successfully!"
    driver.quit
  end
end
```

### Breaking Down the Code

*   **`add_preference`**: While `add_argument` controls how Chrome launches (e.g., `--headless`), `add_preference` modifies the internal Chrome Settings (the stuff you normally configure by clicking Settings > Downloads in the UI).
*   **`File.absolute_path`**: The browser expects an absolute path on your hard drive (e.g., `C:/Projects/Framework/data/users.csv`). Using Ruby's native `File` library guarantees we construct the correct path dynamically, regardless of whether you run the test on Mac, Windows, or Linux.

### Execution Output

```
Blog 22: File Uploads and Downloads
Uploading file: C:/Users/admin/Projects/mcyt-sel-ruby/data/users.csv
File uploaded successfully!
  uploads a file without opening the OS window
Finished in 2.11 seconds
1 example, 0 failures
```

### Conclusion

You have successfully bypassed the two biggest traps that break beginner test automation frameworks! By keeping your interactions strictly within the HTML DOM (`send_keys` to the input tag) and using Chrome Preferences to bypass native OS dialogs, your tests will run flawlessly in CI/CD pipelines without ever getting stuck.

In **Blog 23**, we are going to look at another advanced interaction: **Simulating Complex Mouse and Keyboard Actions using the ActionBuilder API**!
