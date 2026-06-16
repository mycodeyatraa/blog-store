---
title: Dissecting the Payload: Parsing JSON and XML
date: 05-Dec-2024
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [typescript, selenium, api-testing, json, xml, parsing]
category: Selenium TypeScript
categories: [Selenium TypeScript, API Testing Integration]
excerpt: >-
  API data is rarely clean. Master deep array traversals and learn how to convert legacy SOAP XML payloads into readable JSON objects.
readTime: 4 min read
---

# Dissecting the Payload: Parsing JSON and XML

In API Automation, data rarely comes back in a perfectly simple object.

Often, you will receive a massive **JSON Array** of 500 records and you need to find *one specific user* deep inside it.

Alternatively, you may be integrating with a legacy Enterprise backend (like SOAP) that refuses to speak JSON and instead returns ugly **XML** payloads.

In this tutorial, we learn how to traverse arrays and dynamically convert XML into JSON!

---

## 1. Traversing Deep JSON Arrays

JavaScript (and TypeScript) natively handles JSON beautifully. Let's look at how to use standard array methods like `.find()` to navigate a massive payload, and how to use dot notation to drill down into nested objects.

Create `tests/api_parsing.test.ts`:

```typescript
import { ApiUtils } from "./utils/ApiUtils";
describe("Phase 5 - API Testing: Parsing Data", () => {
  it("Should traverse and parse a complex JSON response", async () => {
    // 1. Fetching a JSON Array of users
    const response = await ApiUtils.get("/users");
    const users = response.data;
    // 2. Validate it's an array
    expect(Array.isArray(users)).toBe(true);
    // 3. Find a specific user in the array (e.g. Samantha)
    const targetUser = users.find((user: any) => user.username === "Samantha");
    expect(targetUser).toBeDefined();
    // 4. Traverse deep nested JSON objects (user.address.geo.lat)
    const latitude = targetUser.address.geo.lat;
    expect(latitude).toBe("-68.6102");
    console.log(`Successfully parsed deeply nested JSON! Samantha's latitude is ${latitude}`);
  });
});
```

---

## 2. Converting Legacy XML to JSON

Working with raw XML in JavaScript is painful. It is heavily nested and lacks dot-notation accessibility. 

The industry standard approach is to use the `xml2js` library to instantly convert the XML payload into a JSON object so we can use standard JS features on it!

First, install the library:

```bash
npm install xml2js
npm install --save-dev @types/xml2js
```

Now, let's update our test file to convert a mock XML string:

```typescript
import { parseStringPromise } from "xml2js";
describe("Phase 5 - API Testing: Parsing Data", () => {
  // ... previous test
  it("Should convert and parse an XML response into JSON", async () => {
    // 1. We mock an XML payload that a legacy SOAP backend might return
    const mockXml = `
      <User>
        <Id>105</Id>
        <Username>LegacyAdmin</Username>
        <Role>Manager</Role>
      </User>
    `;
    // 2. Parse the XML string into a usable JSON object!
    const jsonResult = await parseStringPromise(mockXml, { explicitArray: false });
    // 3. Validate we can now interact with it exactly like JSON
    expect(jsonResult.User.Id).toBe("105");
    expect(jsonResult.User.Username).toBe("LegacyAdmin");
    console.log(`Successfully converted XML to JSON! Username: ${jsonResult.User.Username}`);
  });
});
```

*(Note: `explicitArray: false` tells the parser to turn single-element XML nodes into flat strings, rather than arrays of length 1, making it much easier to read!)*

---

## 3. Test Execution Output

```bash
> jest tests/api_parsing.test.ts
```

Output:

```text
 PASS  tests/api_parsing.test.ts
  Phase 5 - API Testing: Parsing Data
    √ Should traverse and parse a complex JSON response (145 ms)
    √ Should convert and parse an XML response into JSON (12 ms)
  console.log
    Successfully parsed deeply nested JSON! Samantha's latitude is -68.6102
  console.log
    Successfully converted XML to JSON! Username: LegacyAdmin
```

## Conclusion

By mastering `.find()`, `.filter()`, and `xml2js`, you guarantee that no matter how complex or archaic the backend's payload is, you can extract exactly what you need to drive your automation logic!

We are almost finished. There is only **one tutorial left** in the entire TypeScript series: **API Contract Validations using JSON Schema!**
