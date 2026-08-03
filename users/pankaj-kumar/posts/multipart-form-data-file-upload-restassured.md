---
title: Multi-part Form Data & File Upload in REST-Assured
date: 19-Feb-2026
lastUpdated: 19-Feb-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [restassured, java, file-upload, multipart, api-testing]
category: REST-Assured
categories: [REST-Assured, API Testing, Java]
excerpt: >-
  Automate file upload endpoints easily! Learn how to send multipart form-data requests and verify file downloads with REST-Assured.
readTime: 7 min read
---

# Multi-part Form Data & File Upload in REST-Assured

Testing endpoints that accept documents, images, or CSV reports requires handling `multipart/form-data` requests rather than standard `application/json` payloads.

REST-Assured provides native `.multiPart()` methods to attach files, binary streams, and form fields effortlessly.

---

## 1. Anatomy of a Multipart Request

A multipart request splits the HTTP payload into distinct boundary-separated sections, each containing headers like `Content-Disposition: form-data; name="file"; filename="report.pdf"`.

---

## 2. Automated File Upload Test Suite

Below is a complete Java test demonstrating file uploads and binary file downloads:

**src/test/java/com/mycodeyatra/FileUploadTest.java**

```java
package com.mycodeyatra;
 
import io.restassured.RestAssured;
import io.restassured.http.ContentType;
import org.junit.jupiter.api.BeforeAll;
import org.junit.jupiter.api.Test;
 
import java.io.File;
 
import static io.restassured.RestAssured.given;
import static org.hamcrest.MatcherAssert.assertThat;
import static org.hamcrest.Matchers.*;
 
public class FileUploadTest {
 
    @BeforeAll
    public static void setup() {
        RestAssured.baseURI = "https://mycodeyatra.com/api";
    }
 
    @Test
    public void testFileUploadEndpoint() {
        File fileToUpload = new File("src/test/resources/sample-doc.pdf");
 
        given()
            .contentType(ContentType.MULTIPART)
            .multiPart("file", fileToUpload, "application/pdf") // Key name, File object, MIME type
            .formParam("description", "Q1 Test Report")
        .when()
            .post("/documents/upload")
        .then()
            .statusCode(200)
            .body("status", equalTo("SUCCESS"))
            .body("fileName", equalTo("sample-doc.pdf"))
            .body("fileSize", greaterThan(0));
    }
 
    @Test
    public void testFileDownloadVerification() {
        byte[] downloadedBytes = given()
        .when()
            .get("/documents/download/101")
        .then()
            .statusCode(200)
            .extract()
            .asByteArray();
 
        assertThat(downloadedBytes.length, greaterThan(0));
    }
}
```

---

## Best Practices

1. **Store Test Artifacts in Resources**: Keep test sample files inside `src/test/resources/` so tests remain platform-independent.
2. **Set Explicit MIME Types**: Always pass explicit MIME types (e.g., `image/png`, `application/pdf`) to mimic true browser behavior.
