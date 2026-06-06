---
title: Parallel Execution & Generating Cucumber Reports
date: 15-Mar-2025
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [karate, bdd, parallel, reporting, cucumber]
category: API Testing
categories: [API Testing, BDD, Java, Automation, Performance]
excerpt: >-
  Supercharge your test suite! Learn how to run your Karate API tests concurrently across multiple threads and generate stunning visual HTML reports.
readTime: 4 min read
---

# Blog #9: Parallel Execution & Generating Cucumber Reports

Welcome to Part 9 of the **Karate API Testing Series**!

As your test suite scales from 10 tests to 10,000 tests, execution time becomes the most critical bottleneck in your CI/CD pipeline. Sequential execution is simply not viable for modern engineering teams.

Karate solves this beautifully. It is completely thread-safe by default, allowing you to run all your feature files concurrently. Let's write a parallel runner and hook it up to an open-source library (`cucumber-reporting`) to generate stunning HTML reports.

## 1. Updating Dependencies
To generate beautiful, interactive reports, add the `cucumber-reporting` dependency to your `pom.xml`:

```xml
<dependency>
    <groupId>net.masterthought</groupId>
    <artifactId>cucumber-reporting</artifactId>
    <version>5.7.5</version>
    <scope>test</scope>
</dependency>
```

## 2. The Parallel JUnit Runner
Instead of executing feature files one by one, we will use the `Runner.path(...).parallel(...)` builder. 

Here is our new `ParallelRunner.java`:

```java
// src/test/java/com/mycodeyatra/karate/ParallelRunner.java
package com.mycodeyatra.karate;
import com.intuit.karate.Results;
import com.intuit.karate.Runner;
import static org.junit.jupiter.api.Assertions.*;
import org.junit.jupiter.api.Test;
import java.io.File;
import java.util.ArrayList;
import java.util.Collection;
import java.util.List;
import net.masterthought.cucumber.Configuration;
import net.masterthought.cucumber.ReportBuilder;
import org.apache.commons.io.FileUtils;
class ParallelRunner {
    @Test
    void testParallel() {
        // Run all feature files in parallel with 5 threads
        Results results = Runner.path("classpath:com/mycodeyatra/karate")
                .outputCucumberJson(true)
                .parallel(5);
        generateReport(results.getReportDir());
        // Assert that there are zero failures across all threads
        assertEquals(0, results.getFailCount(), results.getErrorMessages());
    }
    public static void generateReport(String karateOutputPath) {
        Collection<File> jsonFiles = FileUtils.listFiles(new File(karateOutputPath), new String[] {"json"}, true);
        List<String> jsonPaths = new ArrayList<>();
        jsonFiles.forEach(file -> jsonPaths.add(file.getAbsolutePath()));
        Configuration config = new Configuration(new File("target"), "mcyt-api-karate");
        ReportBuilder reportBuilder = new ReportBuilder(jsonPaths, config);
        reportBuilder.generateReports();
    }
}
```

## 3. Execution and Reports!
When we run `mvn clean test -Dtest=ParallelRunner`, Karate spawns 5 threads and blasts through our entire project simultaneously. 

Look at this execution log:

```bash
======================================================
elapsed:  15.13 | threads:    5 | thread time: 13.16 
features:     8 | skipped:    1 | efficiency: 0.17
scenarios:   11 | passed:    11 | failed: 0
======================================================
Jun 07, 2026 4:47:08 AM net.masterthought.cucumber.ReportParser parseJsonFiles
INFO: File '...\target\karate-reports\com.mycodeyatra.karate.ui.ui.json' contains 1 feature(s)
[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 19.738 s - in com.mycodeyatra.karate.ParallelRunner
[INFO] BUILD SUCCESS
```

If you navigate to `target/cucumber-html-reports/overview-features.html`, you will be greeted by a gorgeous, fully interactive UI displaying charts, execution times, and step-by-step logs of your parallel run.

We are at the finish line! In our next and final blog, we will master **CI/CD Integration and Mocking with Karate**!
