---
title: Reading the Unreadable: Parsing PDF Documents in Selenium
date: 06-Aug-2026
lastUpdated: 06-Aug-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [java, selenium, pdf-validation, pdfbox, file-downloads, test-automation, backend-validation]
category: Selenium Java
categories: [Selenium Java, Enterprise Validation]
excerpt: >-
  A downloaded file is not a black box. Learn how to configure ChromeDriver to download files silently, and use Apache PDFBox to extract and assert textual data natively from binary PDF invoices.
readTime: 6 min read
---

# Reading the Unreadable: Parsing PDF Documents in Selenium

In enterprise applications, data is not always displayed cleanly on a webpage. Applications frequently generate and download binary files. 

Imagine a banking application where a user clicks "Download Bank Statement". The browser downloads a PDF file to the local machine. If your Selenium test stops at verifying the "Download Successful" toast message, you have a massive blind spot. What if the backend PDF generation microservice failed, and the downloaded PDF is completely blank or contains the wrong account balance?

In this tutorial, we will learn how to break into binary files using **Apache PDFBox** to extract text from downloaded PDFs and assert their contents directly within our Selenium tests!

---

## 1. Introducing Apache PDFBox

A PDF is not an HTML document. You cannot inspect it with DevTools, and you cannot use XPath or CSS Selectors to find text inside of it. A PDF is a complex, compiled binary format.

To parse it, we need a specialized library. **Apache PDFBox** is the industry standard open-source Java tool for working with PDF documents.

Add the dependency to your `pom.xml`:

```xml
<dependency>
    <groupId>org.apache.pdfbox</groupId>
    <artifactId>pdfbox</artifactId>
    <version>3.0.1</version>
</dependency>
```

---

## 2. Configuring Chrome to Download Silently

By default, when Selenium clicks a download link, Chrome might prompt the user with a "Save As" dialog box. This native OS dialog cannot be interacted with via Selenium, causing the test to hang indefinitely.

Before we write the PDF parsing code, we must configure our `ChromeDriver` to silently download files to a specific directory without prompting.

```java
import org.openqa.selenium.chrome.ChromeOptions;
import java.util.HashMap;
import java.util.Map;
public ChromeOptions getDownloadOptions(String downloadPath) {
    Map<String, Object> prefs = new HashMap<>();
    // 0 means disable the 'Save As' prompt
    prefs.put("download.prompt_for_download", false);
    // Set the default download directory
    prefs.put("download.default_directory", downloadPath);
    // Disable the PDF Viewer plugin so PDFs download instead of opening in a new tab
    prefs.put("plugins.always_open_pdf_externally", true);
    ChromeOptions options = new ChromeOptions();
    options.setExperimentalOption("prefs", prefs);
    return options;
}
```

---

## 3. Writing the PDF Validation Test

Let's write a complete End-to-End test. 

We will configure Chrome to download files to a temporary `target/downloads` folder. We will use Selenium to log into an e-commerce site and download an Order Invoice. Finally, we will use PDFBox to load the file into memory, extract all the text, and assert that the correct `$Total` and `Order ID` are printed on the invoice!

```java
import org.apache.pdfbox.pdmodel.PDDocument;
import org.apache.pdfbox.text.PDFTextStripper;
import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.chrome.ChromeOptions;
import org.testng.Assert;
import org.testng.annotations.AfterMethod;
import org.testng.annotations.BeforeMethod;
import org.testng.annotations.Test;
import java.io.File;
import java.io.IOException;
public class PdfValidationTest {
    WebDriver driver;
    String downloadDir = System.getProperty("user.dir") + "/target/downloads";
    @BeforeMethod
    public void setup() {
        // Ensure the download directory exists
        new File(downloadDir).mkdirs();
        // Pass the custom download preferences to Chrome
        ChromeOptions options = getDownloadOptions(downloadDir);
        driver = new ChromeDriver(options);
    }
    @Test
    public void testInvoicePdfContents() throws Exception {
        String expectedOrderId = "ORD-998877";
        String expectedTotal = "$1,250.00";
        // --- PART 1: The UI Download Workflow ---
        driver.get("https://practice.mycodeyatra.com/orders");
        // Click the download button for a specific order
        driver.findElement(By.id("download-invoice-" + expectedOrderId)).click();
        // Wait for the file to download (Custom wait logic recommended here)
        Thread.sleep(3000); 
        // --- PART 2: The PDF Validation ---
        // 1. Locate the downloaded file on the local file system
        File downloadedPdf = new File(downloadDir + "/invoice_" + expectedOrderId + ".pdf");
        Assert.assertTrue(downloadedPdf.exists(), "The PDF file failed to download!");
        // 2. Load the binary PDF into Apache PDFBox
        try (PDDocument document = PDDocument.load(downloadedPdf)) {
            // 3. Initialize the Text Stripper
            PDFTextStripper pdfStripper = new PDFTextStripper();
            // Optional: You can extract text from specific pages!
            // pdfStripper.setStartPage(1);
            // pdfStripper.setEndPage(1);
            // 4. Extract all text from the PDF into a Java String
            String pdfText = pdfStripper.getText(document);
            System.out.println("--- EXTRACTED PDF TEXT ---");
            System.out.println(pdfText);
            System.out.println("--------------------------");
            // 5. Assert that the business logic was rendered onto the document!
            Assert.assertTrue(pdfText.contains("Invoice Number: " + expectedOrderId), 
                "The Order ID is missing from the PDF!");
            Assert.assertTrue(pdfText.contains("Total Due: " + expectedTotal), 
                "The Financial Total is incorrect on the PDF!");
        } catch (IOException e) {
            Assert.fail("Failed to parse the PDF document: " + e.getMessage());
        } finally {
            // Clean up: Delete the file after the test finishes
            downloadedPdf.delete();
        }
    }
    @AfterMethod
    public void teardown() {
        driver.quit();
    }
}
```

---

## 4. Limitations of PDFBox

It is crucial to understand what PDFBox *can* and *cannot* do.

Apache PDFBox extracts **embedded text**. If your application generates a PDF that is essentially a giant scanned image (like a JPEG embedded inside a PDF wrapper), PDFBox will return a blank string because there is no selectable text layer!

To extract text from image-based PDFs, you must bypass PDFBox entirely and use **OCR (Optical Character Recognition)** libraries, such as `Tesseract OCR`.

## Conclusion

A downloaded file is not a black box. By configuring Chrome preferences to handle downloads silently and utilizing powerful binary parsers like Apache PDFBox, you can extend your End-to-End test coverage straight onto the local file system.

But what if the downloaded file isn't a PDF? What if it's a massive, multi-tabbed Financial Report exported as an `.xlsx` file? 

In our next tutorial, we will learn how to ditch PDFBox and integrate **Apache POI** to parse and validate downloaded Excel spreadsheets natively within Selenium!
