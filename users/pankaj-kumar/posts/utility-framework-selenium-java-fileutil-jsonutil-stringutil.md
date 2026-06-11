---
title: Utility Framework in Selenium Java: Building Reusable FileUtil, JsonUtil and StringUtil
date: 28-Jul-2024
lastUpdated: 11-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [selenium, java, utility, fileutil, jsonutil, stringutil, framework]
category: UI Automation
categories: [UI Automation, Selenium, Java]
excerpt: >-
  Build a professional Selenium utility layer in Java with FileUtil, JsonUtil, and StringUtil. Learn how to eliminate boilerplate file, JSON, and string operations from test classes using clean, reusable helpers.
readTime: 6 min read
---

﻿# Utility Framework in Selenium Java: Building Reusable FileUtil, JsonUtil & StringUtil

A professional Selenium framework is never just about Page Objects and driver management. The **utility layer** is the invisible backbone that eliminates repetitive boilerplate — file reads, JSON parsing, string manipulation — from every test class. In this blog, we build three focused, reusable utility classes that your entire framework will depend on.

---

## What You Will Build

| Utility Class | Responsibility |
|---|---|
| `FileUtil` | Read/write files, check existence, extract extensions |
| `JsonUtil` | Parse JSON files and strings, serialise objects |
| `StringUtil` | Blank check, truncate, slug generation, title case |
| `UtilityFrameworkTest` | Validates all three utilities with TestNG assertions |

---

## Project Structure

All utility classes live under `src/main/java/com/mycodeyatra/utils/` so they are available to both production support code and test code:

![diagram_1](https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/posts/utility-framework-selenium-java-fileutil-jsonutil-stringutil/images/diagram_1.png)

---

## Step-by-Step Code Walkthrough

### 1. FileUtil (FileUtil.java)

`FileUtil` wraps `java.nio.file.Files` to provide clean, exception-free file operations. No try-catch blocks scattered across test classes.

```java
package com.mycodeyatra.utils;
import java.io.IOException;
import java.nio.charset.StandardCharsets;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.Paths;
public class FileUtil {
 private FileUtil() {}
 public static String readFile(String filePath) {
  try {
   return Files.readString(Paths.get(filePath), StandardCharsets.UTF_8);
  } catch (IOException e) {
   throw new RuntimeException("FileUtil: Failed to read file -> " + filePath, e);
  }
 }
 public static void writeFile(String filePath, String content) {
  try {
   Path path = Paths.get(filePath);
   Files.createDirectories(path.getParent());
   Files.writeString(path, content, StandardCharsets.UTF_8);
  } catch (IOException e) {
   throw new RuntimeException("FileUtil: Failed to write file -> " + filePath, e);
  }
 }
 public static boolean fileExists(String filePath) {
  return Files.exists(Paths.get(filePath));
 }
 public static String getExtension(String filePath) {
  String name = Paths.get(filePath).getFileName().toString();
  int dot = name.lastIndexOf('.');
  return (dot > 0) ? name.substring(dot + 1) : "";
 }
}
```

### 2. JsonUtil (JsonUtil.java)

`JsonUtil` wraps Jackson `ObjectMapper` to give clean, one-liner JSON parsing without checked exceptions in tests.

```java
package com.mycodeyatra.utils;
import com.fasterxml.jackson.core.type.TypeReference;
import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.ObjectMapper;
import java.io.File;
import java.io.IOException;
import java.util.List;
import java.util.Map;
public class JsonUtil {
 private JsonUtil() {}
 private static final ObjectMapper MAPPER = new ObjectMapper();
 public static <T> T fromFile(String filePath, Class<T> clazz) {
  try {
   return MAPPER.readValue(new File(filePath), clazz);
  } catch (IOException e) {
   throw new RuntimeException("JsonUtil: Failed to parse JSON file -> " + filePath, e);
  }
 }
 public static List<Map<String, Object>> fromFileAsList(String filePath) {
  try {
   return MAPPER.readValue(new File(filePath), new TypeReference<>() {});
  } catch (IOException e) {
   throw new RuntimeException("JsonUtil: Failed to parse JSON array -> " + filePath, e);
  }
 }
 public static JsonNode parseString(String json) {
  try {
   return MAPPER.readTree(json);
  } catch (IOException e) {
   throw new RuntimeException("JsonUtil: Failed to parse JSON string", e);
  }
 }
 public static String toJson(Object object) {
  try {
   return MAPPER.writerWithDefaultPrettyPrinter().writeValueAsString(object);
  } catch (IOException e) {
   throw new RuntimeException("JsonUtil: Failed to serialise object to JSON", e);
  }
 }
}
```

### 3. StringUtil (StringUtil.java)

`StringUtil` handles all the string manipulation patterns that appear repeatedly across test data preparation, assertion messages, and report generation.

```java
package com.mycodeyatra.utils;
public class StringUtil {
 private StringUtil() {}
 public static boolean isBlank(String value) {
  return value == null || value.trim().isEmpty();
 }
 public static String truncate(String value, int maxLen) {
  if (value == null) return "";
  return value.length() <= maxLen ? value : value.substring(0, maxLen) + "...";
 }
 public static String removeWhitespace(String value) {
  return (value == null) ? "" : value.replaceAll("\\s+", "");
 }
 public static String toSlug(String value) {
  if (value == null) return "";
  return value.trim().toLowerCase()
   .replaceAll("[^a-z0-9\\s-]", "")
   .replaceAll("\\s+", "-");
 }
 public static String toTitleCase(String value) {
  if (isBlank(value)) return "";
  String[] words = value.trim().split("\\s+");
  StringBuilder sb = new StringBuilder();
  for (String word : words) {
   if (!word.isEmpty()) {
    sb.append(Character.toUpperCase(word.charAt(0)))
     .append(word.substring(1).toLowerCase())
     .append(" ");
   }
  }
  return sb.toString().trim();
 }
}
```

### 4. Validation Test Suite (UtilityFrameworkTest.java)

```java
package com.mycodeyatra.tests;
import com.mycodeyatra.utils.FileUtil;
import com.mycodeyatra.utils.JsonUtil;
import com.mycodeyatra.utils.StringUtil;
import org.testng.Assert;
import org.testng.annotations.Test;
import java.util.Map;
public class UtilityFrameworkTest {
 private static final String TEMP_FILE = "target/test-output/util_test.txt";
 @Test(description = "FileUtil: write and read back a file")
 public void testFileWriteAndRead() {
  String content = "Hello from FileUtil!";
  FileUtil.writeFile(TEMP_FILE, content);
  String read = FileUtil.readFile(TEMP_FILE);
  Assert.assertEquals(read, content, "File read content should match written content");
  System.out.println("[FileUtil] Write + Read PASSED: " + read);
 }
 @Test(description = "FileUtil: fileExists returns correct boolean")
 public void testFileExists() {
  FileUtil.writeFile(TEMP_FILE, "exists check");
  Assert.assertTrue(FileUtil.fileExists(TEMP_FILE));
  Assert.assertFalse(FileUtil.fileExists("target/non_existent.txt"));
  System.out.println("[FileUtil] fileExists PASSED");
 }
 @Test(description = "FileUtil: getExtension extracts file extension correctly")
 public void testGetExtension() {
  Assert.assertEquals(FileUtil.getExtension("data/users.json"), "json");
  Assert.assertEquals(FileUtil.getExtension("reports/result.xlsx"), "xlsx");
  Assert.assertEquals(FileUtil.getExtension("README"), "");
  System.out.println("[FileUtil] getExtension PASSED");
 }
 @Test(description = "JsonUtil: parse JSON string into JsonNode")
 public void testParseJsonString() {
  String json = "{\"username\":\"admin\",\"role\":\"tester\"}";
  var node = JsonUtil.parseString(json);
  Assert.assertEquals(node.get("username").asText(), "admin");
  System.out.println("[JsonUtil] parseString PASSED: username=" + node.get("username").asText());
 }
 @Test(description = "JsonUtil: serialise object to JSON string")
 public void testToJson() {
  Map<String, String> map = Map.of("env", "staging", "browser", "chrome");
  String json = JsonUtil.toJson(map);
  Assert.assertTrue(json.contains("staging"));
  System.out.println("[JsonUtil] toJson PASSED: " + json);
 }
 @Test(description = "StringUtil: isBlank detects null and whitespace")
 public void testIsBlank() {
  Assert.assertTrue(StringUtil.isBlank(null));
  Assert.assertTrue(StringUtil.isBlank("   "));
  Assert.assertFalse(StringUtil.isBlank("hello"));
  System.out.println("[StringUtil] isBlank PASSED");
 }
 @Test(description = "StringUtil: truncate limits string length")
 public void testTruncate() {
  Assert.assertEquals(StringUtil.truncate("Hello World", 5), "Hello...");
  Assert.assertEquals(StringUtil.truncate("Hi", 5), "Hi");
  System.out.println("[StringUtil] truncate PASSED");
 }
 @Test(description = "StringUtil: toSlug converts string to URL slug")
 public void testToSlug() {
  Assert.assertEquals(StringUtil.toSlug("Page Object Model"), "page-object-model");
  System.out.println("[StringUtil] toSlug PASSED");
 }
 @Test(description = "StringUtil: toTitleCase capitalises each word")
 public void testToTitleCase() {
  Assert.assertEquals(StringUtil.toTitleCase("page object model"), "Page Object Model");
  System.out.println("[StringUtil] toTitleCase PASSED");
 }
}
```

---

## When to Use Each Utility

| Scenario | Use |
|---|---|
| Reading a `.properties` or `.txt` config file | `FileUtil.readFile()` |
| Loading test data from a `.json` file | `JsonUtil.fromFileAsList()` |
| Dynamically accessing an API response field | `JsonUtil.parseString()` |
| Sanitising user input before assertion | `StringUtil.removeWhitespace()` |
| Building a slug for a URL or report name | `StringUtil.toSlug()` |
| Checking whether a downloaded file exists | `FileUtil.fileExists()` |

---

## Key Takeaways

1. **Utility classes should be stateless**: Use private constructors and static methods — no instance state, no thread-safety issues.
2. **Wrap checked exceptions**: Convert `IOException` into `RuntimeException` at the utility boundary so test code stays clean.
3. **Single responsibility per class**: `FileUtil` handles files only, `JsonUtil` handles JSON only. Never mix concerns.
4. **Singleton ObjectMapper**: `Jackson ObjectMapper` is expensive to create — always share a single static instance via `JsonUtil`.
5. **Place utilities in `main/java`**: Utilities shared between production support code and test code belong in `src/main`, not `src/test`.
