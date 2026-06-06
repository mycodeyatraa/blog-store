---
title: Testing File Uploads & Downloads
date: 27-Feb-2025
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [supertest, file-upload, file-download, multipart, jest]
category: API Testing
categories: [API Testing, NodeJS, File Handling, Automation]
excerpt: >-
  Master binary payloads and multipart form boundaries. Learn how to effortlessly test file uploads and downloads using SuperTest.
readTime: 4 min read
---

In the age of multimedia apps, file handling is a core feature of most backend systems. Profile pictures, PDF reports, and CSV data exports all require endpoints that deal with binary streams rather than raw JSON strings.

Testing these endpoints manually is tedious. Let's explore how **SuperTest** makes automating file uploads (`multipart/form-data`) and downloads trivial.

---

## 1. Testing File Uploads

When you upload a file via a typical web form, the browser formats the HTTP request as `multipart/form-data`. 

In SuperTest, you don't have to manually construct these complex payload boundaries. You simply use the `.attach()` method! SuperTest will dynamically read the file from your local disk and format the payload perfectly.

```typescript
import request from 'supertest';
import path from 'path';
describe('Testing File Uploads & Downloads', () => {
    const API_URL = 'http://localhost:8080';
    it('1. Upload a File (multipart/form-data)', async () => {
        // Resolve the absolute path to a dummy test file in our repo
        const filePath = path.join(__dirname, 'test-file.txt');
        // SuperTest's .attach() method automatically sets Content-Type to multipart/form-data
        const response = await request(API_URL)
            .post('/api/files/upload')
            .attach('file', filePath); // 'file' is the name of the form field expected by Multer
        expect(response.status).toBe(200);
        expect(response.body.message).toBe('File uploaded successfully');
        expect(response.body.file).toHaveProperty('originalname', 'test-file.txt');
    });
```

## 2. Testing File Downloads

When an API returns a file (like a CSV export), it is sending binary data, not JSON. If you try to assert against `response.body` normally, it will fail because SuperTest tries to parse it as an object.

To fix this, we instruct SuperTest to treat the incoming payload as a binary buffer using `.responseType('blob')`.

```typescript
    it('2. Download a File and verify contents', async () => {
        // We instruct SuperTest to parse the binary buffer correctly
        const response = await request(API_URL)
            .get('/api/files/download')
            .responseType('blob'); // Crucial for file downloads!
        // Assert that the headers correctly instruct the browser to download the file
        expect(response.status).toBe(200);
        expect(response.headers['content-type']).toContain('text/csv');
        expect(response.headers['content-disposition']).toContain('attachment; filename="dummy_data.csv"');
        // We can parse the downloaded buffer into a string and assert its contents!
        const fileContents = response.body.toString('utf-8');
        expect(fileContents).toContain('Alice,Admin');
    });
});
```

## 3. Execution Results

Let's execute `npx jest tests/files.test.ts`. Notice how quickly SuperTest streams the data to and from the local file system!

```bash
PASS tests/files.test.ts
  Testing File Uploads & Downloads
    [PASS] 1. Upload a File (multipart/form-data) (72 ms)
    [PASS] 2. Download a File and verify contents (7 ms)
Test Suites: 1 passed, 1 total
Tests:       2 passed, 2 total
Snapshots:   0 total
Time:        3.586 s
```

## 4. Conclusion

Handling files is often considered one of the hardest parts of API automation, but SuperTest abstract away the complexity of multipart boundaries and binary buffers.

Next, we will look into one of the most critical aspects of API testing: **Handling Request Timeouts & Chaos Testing**!
