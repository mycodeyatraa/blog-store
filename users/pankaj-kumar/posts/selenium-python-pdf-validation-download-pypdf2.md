---
title: Validating Invoices: PDF Validation in Selenium Python
date: 24-Mar-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [python, selenium, pdf-validation, file-download, pypdf2, test-automation]
category: Selenium Python
categories: [Selenium Python, Enterprise Validation]
excerpt: >-
  When testing E-Commerce or FinTech apps, clicking 'Download' isn't enough. Learn how to configure Chrome preferences to force-download PDFs and use PyPDF2 to extract and assert internal document text.
readTime: 4 min read
---

# Validating Invoices: PDF Validation in Selenium Python

In the Enterprise world, web applications don't just display data on screen; they generate massive, complex documents. 

If you are automating an E-Commerce site, it is not enough to verify that the "Download Invoice" button works. You must actually download the generated `.pdf` file, open it in the background, and use Python to verify that the correct tax amount and Order ID are printed inside the document!

In this tutorial, we will learn how to configure Selenium to download files automatically, and how to use Python's `PyPDF2` library to extract and assert text from PDF files.

---

## 1. Configuring Chrome to Download Automatically

By default, when you click a PDF link in Chrome, the browser attempts to render it in a new tab using the built-in PDF Viewer. 

For test automation, we do NOT want this. We want the file to immediately download to a specific folder on our hard drive without prompting the user.

We can achieve this by passing experimental preferences to `webdriver.ChromeOptions()`:

```python
import os
from selenium import webdriver
# Define an absolute path for our downloads folder
download_dir = os.path.abspath("test_downloads")
os.makedirs(download_dir, exist_ok=True)
options = webdriver.ChromeOptions()
# Configure Chrome to download automatically
prefs = {
    "download.default_directory": download_dir,
    "download.prompt_for_download": False,
    "download.directory_upgrade": True,
    "plugins.always_open_pdf_externally": True  # Crucial for PDFs!
}
options.add_experimental_option("prefs", prefs)
driver = webdriver.Chrome(options=options)
```

With these options, any click on a `.pdf` link will instantly save the file to our `test_downloads` folder!

---

## 2. Waiting for the Download to Complete

When Selenium clicks a download button, the click command returns instantly, but the file might take several seconds to generate and download. 

If we immediately try to parse the file, Python will throw a `FileNotFoundError`! We need to write a utility function that waits until the file fully appears on disk.

```python
import time
def wait_for_file(directory, filename, timeout=30):
    """
    Polls the directory until the file exists and is not a temporary Chrome download file.
    """
    file_path = os.path.join(directory, filename)
    end_time = time.time() + timeout
    while time.time() < end_time:
        # Chrome creates a .crdownload file while the download is in progress
        crdownload_path = file_path + ".crdownload"
        if os.path.exists(file_path) and not os.path.exists(crdownload_path):
            print(f"✅ Download complete: {file_path}")
            return file_path
        time.sleep(1)
    raise Exception(f"Timeout waiting for file: {filename}")
```

---

## 3. Parsing the PDF with PyPDF2

Now that we have the file on our hard drive, we need to read it! We will use the `PyPDF2` library.

Install it via pip:

```bash
pip install PyPDF2
```

Now, let's write a utility to extract all the text from the document:

```python
from PyPDF2 import PdfReader
def extract_pdf_text(file_path):
    """
    Opens a PDF and extracts text from all pages into a single string.
    """
    text_content = ""
    with open(file_path, "rb") as file:
        reader = PdfReader(file)
        for page in reader.pages:
            text_content += page.extract_text() + "\n"
    return text_content
```

---

## 4. Putting it all together: The Hybrid Test

Let's combine everything into a beautiful, Enterprise-grade test.

```python
def test_invoice_generation():
    # 1. Setup Chrome (using the options from Step 1)
    # driver = ...
    # 2. Trigger the download via Selenium
    driver.get("https://practice.mycodeyatra.com/orders/12345")
    driver.find_element("id", "download-invoice-btn").click()
    # 3. Wait for the file to arrive in our custom folder
    pdf_path = wait_for_file(download_dir, "Invoice_12345.pdf")
    # 4. Extract the text using PyPDF2
    pdf_text = extract_pdf_text(pdf_path)
    # 5. Assert Business Logic!
    print("Validating PDF contents...")
    assert "Invoice #12345" in pdf_text
    assert "Tax: $12.50" in pdf_text
    assert "Total: $137.50" in pdf_text
    print("✅ PDF successfully validated!")
    # Clean up
    os.remove(pdf_path)
    driver.quit()
```

## Conclusion

Automating file downloads and PDF parsing is an incredibly common requirement in FinTech, Healthcare, and E-Commerce domains. By configuring Chrome's experimental preferences and leveraging Python's `PyPDF2` library, you can easily validate complex document generation pipelines!

In the next tutorial, we will tackle the final document type: **Excel Validation**. We will learn how to download `.xlsx` reports and use `pandas` and `openpyxl` to validate massive spreadsheets!
