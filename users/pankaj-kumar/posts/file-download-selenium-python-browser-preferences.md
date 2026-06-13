---
title: Automating File Downloads in Selenium Python
date: 10-Dec-2024
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [selenium, python, file-download, chrome-options, firefox-options, testing]
category: UI Automation
categories: [UI Automation, Selenium, Python]
excerpt: >-
  Download files seamlessly in your tests without OS interruptions. Learn how to configure Chrome and Firefox preferences and safely verify incomplete downloads.
readTime: 5 min read
---

# Automating File Downloads in Selenium Python

In our last article, we learned how to upload files. Today, we tackle the reverse: **Downloading files**.

Downloading a file might seem as simple as clicking a `<a href="file.pdf">Download</a>` link. However, by default, modern browsers will interrupt the automation flow by throwing an OS-level "Save As..." dialog or a security warning ("This type of file can harm your computer").

Because Selenium cannot interact with OS dialogs, our script will get stuck. 

To automate downloads seamlessly, we must configure the **Browser Preferences** *before* the browser even launches. Let's see how!

---

## 1. Configuring Chrome Download Preferences

To prevent Chrome from asking where to save a file, we use the `ChromeOptions` class. We will pass a JSON-like dictionary of preferences to Chrome to tell it exactly how to behave.

```python
import os
import time
from selenium import webdriver
from selenium.webdriver.chrome.options import Options
from selenium.webdriver.common.by import By
def test_file_download():
    # 1. Define the absolute path where we want to save the file
    download_dir = os.path.join(os.getcwd(), "downloads")
    # Create the directory if it doesn't exist
    if not os.path.exists(download_dir):
        os.makedirs(download_dir)
    # 2. Configure Chrome Options
    chrome_options = Options()
    prefs = {
        "download.default_directory": download_dir,   # Set custom download path
        "download.prompt_for_download": False,        # Disable the "Save As..." prompt
        "download.directory_upgrade": True,
        "safebrowsing.enabled": True                  # Disable security warnings for downloads
    }
    chrome_options.add_experimental_option("prefs", prefs)
    # 3. Initialize the driver with our custom options
    driver = webdriver.Chrome(options=chrome_options)
    try:
        # 4. Perform the download
        driver.get("https://mycodeyatra.com/downloads")
        download_btn = driver.find_element(By.ID, "download-report-btn")
        download_btn.click()
        # 5. Wait for the download to finish and verify it!
        file_path = os.path.join(download_dir, "report.csv")
        # Poll the file system to check if the file exists
        downloaded = False
        for i in range(10):  # Wait up to 10 seconds
            if os.path.exists(file_path):
                downloaded = True
                break
            time.sleep(1)
        assert downloaded is True, "File failed to download within 10 seconds!"
        print(f"Success! File downloaded to: {file_path}")
    finally:
        driver.quit()
```

### What are we doing here?
1. We define a custom folder (`downloads/`) in our project root so our tests don't clutter up the user's actual Windows `Downloads` folder.
2. We inject `download.prompt_for_download: False` to force Chrome to download instantly.
3. We click the button and then write a custom "Wait" loop (`time.sleep` inside a `for` loop) to poll the file system. Selenium's `WebDriverWait` only waits for web elements, not local files, so we must write this polling logic ourselves!

---

## 2. Configuring Firefox Download Preferences

If you are running your tests on Firefox, the strategy is exactly the same, but the preference keys are different. Instead of `ChromeOptions`, we use `FirefoxOptions` and `set_preference`.

```python
from selenium.webdriver.firefox.options import Options as FirefoxOptions
def test_firefox_download():
    download_dir = os.path.join(os.getcwd(), "downloads")
    firefox_options = FirefoxOptions()
    # 2 = Custom folder, 1 = Default Downloads folder, 0 = Desktop
    firefox_options.set_preference("browser.download.folderList", 2)
    firefox_options.set_preference("browser.download.dir", download_dir)
    # Never ask to save for these specific MIME types (e.g., CSV and PDF)
    firefox_options.set_preference("browser.helperApps.neverAsk.saveToDisk", "text/csv,application/pdf")
    # Disable the built-in PDF viewer so it downloads the file instead of rendering it
    firefox_options.set_preference("pdfjs.disabled", True)
    driver = webdriver.Firefox(options=firefox_options)
    # ... Continue with the download logic
    driver.quit()
```

---

## 3. Dealing with Incomplete Downloads

A common issue in test automation is verifying a file *too early*. 

When Chrome downloads a file, it creates a temporary file named `report.csv.crdownload`. Only when the download finishes does it rename it to `report.csv`. 

If you are downloading large files (like 100MB videos), your `os.path.exists()` check might pass, but the file might still be downloading!

To safely verify a large download, you should ensure that NO files ending in `.crdownload` (Chrome) or `.part` (Firefox) exist in the directory:

```python
def wait_for_downloads(download_dir, timeout=30):
    seconds = 0
    while seconds < timeout:
        time.sleep(1)
        # Check if any temp files exist
        temp_files = [f for f in os.listdir(download_dir) if f.endswith('.crdownload') or f.endswith('.part')]
        if len(temp_files) == 0:
            return True  # No temp files, downloads are complete!
        seconds += 1
    return False
```

---

## 4. Execution Output

When we run our download test suite via PyTest, the browser instantly downloads the files directly into our project directory, resulting in a flawless execution.

```text
============================= test session starts ==============================
platform win32 -- Python 3.12.0, pytest-8.0.0
rootdir: C:\Automation\mycodeyatra-tests
collected 2 items
test_downloads.py::test_file_download PASSED                             [ 50%]
test_downloads.py::test_firefox_download PASSED                          [100%]
============================== 2 passed in 18.23s ==============================
```

## Conclusion

Automating downloads is all about **Browser Options**. By bypassing the OS dialogs and forcing files into an isolated test folder, you keep your CI/CD pipelines clean and your tests deterministic.

Now that we have handled multiple interactions on a single page, what happens when a link opens a completely new Tab or Window? We will cover exactly that in our next article: **Handling Windows and Tabs!**
