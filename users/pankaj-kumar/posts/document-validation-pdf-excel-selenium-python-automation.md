---
title: Document Validation: Testing PDFs and Excel Files in Python
date: 10-Jun-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [python, selenium, pdf, excel, validation, pypdf2, openpyxl]
category: Enterprise Validation
categories: [Enterprise Validation, Python, Automation]
excerpt: >-
  Don't just verify that a file downloaded! Learn the ultimate Enterprise strategy: parsing and asserting the internal financial data of PDFs and Excel spreadsheets natively using PyPDF2 and openpyxl.
readTime: 6 min read
---

# Document Validation: Testing PDFs and Excel Files in Python

You have reached the final layer of our Enterprise Validation Strategy. We have verified Databases, GraphQL APIs, Kafka Event Streams, and Emails. But what happens when your application exports a file?

When a user clicks "Download Invoice", standard automation frameworks simply verify that a `.pdf` file appeared in the `Downloads` folder. **This is fundamentally flawed.** What if the PDF is blank? What if the financial totals inside the Excel export are calculated incorrectly?

In this article, we will teach you how to open, parse, and assert the internal text of downloaded PDFs and Excel spreadsheets natively in Python using `PyPDF2` and `openpyxl`.

---

## 1. Validating PDF Invoices

To read PDF documents, we will use the `PyPDF2` library.

### Installation

```bash
pip install PyPDF2
```

### The PDF Test
Assume our Selenium script just clicked the "Download Invoice" button for an order totaling $1,450.00. We will locate the downloaded file and assert the contents.

**tests/test_pdf_validation.py**

```python
import os
import time
from PyPDF2 import PdfReader
from selenium.webdriver.common.by import By
def test_invoice_pdf_contents(driver):
    # 1. UI ACTION: Trigger the Download
    driver.get("https://mycodeyatra.com/orders/latest")
    driver.find_element(By.ID, "download-invoice-btn").click()
    # Wait for the file to download completely
    time.sleep(3) 
    # 2. LOCATE THE FILE
    # In a real framework, you should configure your WebDriver download directory
    # and fetch the most recently created file programmatically.
    download_dir = "/Users/qa/Downloads/"
    pdf_path = os.path.join(download_dir, "invoice_INV-8890.pdf")
    assert os.path.exists(pdf_path), "PDF Invoice failed to download!"
    # 3. DOCUMENT ASSERTION: Parse the PDF
    print(f"\n[Validation] Opening PDF: {pdf_path}")
    with open(pdf_path, 'rb') as file:
        pdf = PdfReader(file)
        # Verify it's not a blank document
        num_pages = len(pdf.pages)
        assert num_pages > 0, "The generated PDF is completely blank!"
        # Extract text from the first page
        first_page = pdf.pages[0]
        pdf_text = first_page.extract_text()
        print("\n--- Extracted PDF Content ---")
        print(pdf_text[:200]) # Print first 200 characters for debugging
        print("-----------------------------\n")
        # 4. CRITICAL ASSERTIONS
        assert "Invoice #INV-8890" in pdf_text, "Incorrect Invoice Number!"
        assert "Total: $1,450.00" in pdf_text, "Financial Total is incorrect!"
        assert "Status: PAID" in pdf_text, "Payment status is wrong!"
```

If the backend PDF generation library fails and renders `Total: NaN`, your Selenium test will catch the bug!

---

## 2. Validating Excel Exports

E-Commerce portals often have "Export to Excel" buttons for accounting departments. To read `.xlsx` files, we use the industry-standard `openpyxl` library.

### Installation

```bash
pip install openpyxl
```

### The Excel Test
Let's assume our Selenium script exported a "Monthly Sales Report". We need to verify that Row 2, Column B contains "March 2025" and that the Total Column is correct.

**tests/test_excel_validation.py**

```python
import os
import time
from openpyxl import load_workbook
from selenium.webdriver.common.by import By
def test_excel_export_contents(driver):
    # 1. UI ACTION: Export the data
    driver.get("https://mycodeyatra.com/admin/reports")
    driver.find_element(By.ID, "export-excel-btn").click()
    time.sleep(3)
    excel_path = "/Users/qa/Downloads/Monthly_Report.xlsx"
    assert os.path.exists(excel_path), "Excel Report failed to download!"
    # 2. DOCUMENT ASSERTION: Parse the Excel File
    print(f"\n[Validation] Opening Excel: {excel_path}")
    # Load the workbook
    workbook = load_workbook(excel_path)
    # Assert the correct sheet was generated
    assert "Sales Data" in workbook.sheetnames, "Missing 'Sales Data' sheet!"
    sheet = workbook["Sales Data"]
    # 3. CELL ASSERTIONS
    # Verify headers
    assert sheet['A1'].value == "Transaction ID"
    assert sheet['B1'].value == "Month"
    assert sheet['C1'].value == "Revenue"
    # Verify data in Row 2
    assert sheet['B2'].value == "March 2025"
    # Verify mathematical accuracy by iterating over columns
    total_revenue = 0
    # Assuming data exists from row 2 to 10
    for row in range(2, 11):
        cell_value = sheet[f'C{row}'].value
        if cell_value is not None:
            total_revenue += float(cell_value)
    expected_total = 55000.00
    assert total_revenue == expected_total, f"Excel data does not add up! Expected {expected_total}, Got {total_revenue}"
```

## Conclusion: The Enterprise Validation Strategy

Congratulations! You have completed Phase 11. You now possess the skills of a Senior SDET. 

A true Enterprise Validation Strategy means testing the **entire system architecture**:
1. **Databases:** Use `SQLAlchemy` (MySQL/PostgreSQL) and `PyMongo` (MongoDB) to verify data persistence.
2. **APIs:** Use `requests` to validate REST and GraphQL mutations.
3. **Event Brokers:** Use `confluent-kafka` to verify asynchronous microservice communication.
4. **Communications:** Use `imaplib` to validate Emails and bypass OTPs.
5. **Documents:** Use `PyPDF2` and `openpyxl` to assert the exact contents of downloaded files.

In Phase 12, we will tackle the final challenge of test automation: **Advanced Reporting and Observability**, transforming raw test failures into beautiful HTML dashboards!
