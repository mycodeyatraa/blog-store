---
title: Handling File Uploads and Downloads
date: 08-Feb-2025
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [restassured, java, multipart, file-upload, file-download]
category: API Testing
categories: [API Testing, Automation, Java]
excerpt: >-
  Discover how to automate file upload forms using multipart/form-data and successfully download and parse CSVs and binary files with RestAssured.
readTime: 6 min read
---

In many modern applications, API testing extends beyond JSON and XML payloads. Often, you will need to automate the uploading of documents or the downloading of reports. 

RestAssured provides native support for `multipart/form-data` file uploads and stream-based downloads!

---

## 1. File Uploads (`multipart/form-data`)

When you upload a file via a browser or Postman, you are typically using a `multipart/form-data` request. RestAssured simulates this using the `.multiPart()` method. 

Let's dynamically create a dummy `.txt` file using standard Java, upload it to our mock server's `/api/files/upload` endpoint, and then clean it up!

```java
@Test(priority = 1)
public void testFileUpload() throws IOException {
    System.out.println("\\n--- Executing File Upload Test ---");
    // 1. Create a temporary dummy file to upload
    File dummyFile = new File("test_upload.txt");
    try (FileWriter writer = new FileWriter(dummyFile)) {
        writer.write("This is a dummy file for RestAssured upload testing.");
    }
    // 2. Execute Multipart form-data upload
    Response response = RestAssured.given()
            .multiPart("file", dummyFile) // 'file' matches the expected form field in the mock API
            .when()
            .post("/api/files/upload")
            .then()
            .extract().response();
    // 3. Validate
    Assert.assertEquals(response.getStatusCode(), 200);
    System.out.println("Upload Response: " + response.jsonPath().getString("message"));
    // Cleanup
    if (dummyFile.exists()) {
        dummyFile.delete();
    }
}
```

Notice the `.multiPart("file", dummyFile)` line? The first parameter `"file"` must exactly match the key expected by the backend API.

## 2. File Downloads

Downloading a file via an API is just a standard GET request! The difference lies in how you handle the *Response*. Instead of parsing JSON, you evaluate the raw bytes or text string of the file itself.

Our Mock Server provides a `/api/files/download` endpoint that returns a CSV file.

```java
@Test(priority = 2)
public void testFileDownload() {
    System.out.println("\\n--- Executing File Download Test ---");
    // Execute GET request to download
    Response response = RestAssured.given()
            .when()
            .get("/api/files/download")
            .then()
            .extract().response();
    // Validate
    Assert.assertEquals(response.getStatusCode(), 200);
    // Assert Content-Type is text/csv
    Assert.assertTrue(response.getContentType().contains("text/csv"));
    // Read file contents as a String (or byte array for binary files)
    String csvContent = response.getBody().asString();
    System.out.println("Downloaded CSV Content:\\n" + csvContent);
    // Validate CSV content
    Assert.assertTrue(csvContent.contains("Alice"));
}
```

If you are downloading binary files (like PDFs or images), you should use `response.getBody().asByteArray()` and write it to a file stream using `FileOutputStream`.

---

## The Execution Output

When we run these tests (`mvn clean test`), here is the console output:

```text
--- Executing File Upload Test ---
Upload Response: File uploaded successfully
--- Executing File Download Test ---
Downloaded CSV Content:
id,name,role
1,Alice,Admin
2,Bob,User
[INFO] Tests run: 2, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 13.91 s -- in TestSuite
```

By mastering `.multiPart()` and evaluating binary payloads, your automation suite can confidently handle complex document workflows!

In the next tutorial, we will explore advanced capabilities like GraphQL and WebSocket testing.
