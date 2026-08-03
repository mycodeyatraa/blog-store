---
title: Parallel Execution & Custom Reporting in Karate
date: 27-Feb-2026
lastUpdated: 27-Feb-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [karate, parallel-execution, reporting, cucumber-reports, ci-cd]
category: API Karate
categories: [API Karate, API Testing, Reporting]
excerpt: >-
  Slash execution times! Scale Karate tests with multi-threaded parallel execution and generate rich Cucumber HTML reports.
readTime: 8 min read
---

# Parallel Execution & Custom Reporting in Karate

As test suites grow to hundreds of API feature files, running tests sequentially creates severe CI/CD bottlenecks.

**Karate DSL** includes built-in parallel execution capabilities powered by Java multi-threading, allowing suite execution times to drop from 30 minutes down to 3 minutes.

---

## 1. Multi-Threaded Parallel Execution Architecture

```
 +------------------------------------------------------------------+
 |                        Karate Test Runner                        |
 +------------------------------------------------------------------+
        |                         |                         |
        v                         v                         v
 [ Thread-1: Users ]     [ Thread-2: Cart ]     [ Thread-3: Payment ]
```

---

## 2. Implementing Parallel Runner & Cucumber HTML Reporter

Configure multi-threaded execution and generate Cucumber HTML dashboards:

**src/test/java/com/mycodeyatra/ParallelTestRunner.java**

```java
package com.mycodeyatra;
 
import com.intuit.karate.Results;
import com.intuit.karate.Runner;
import net.masterthought.cucumber.Configuration;
import net.masterthought.cucumber.ReportBuilder;
import org.apache.commons.io.FileUtils;
import org.junit.jupiter.api.Test;
 
import java.io.File;
import java.util.ArrayList;
import java.util.Collection;
import java.util.List;
 
import static org.junit.jupiter.api.Assertions.assertEquals;
 
public class ParallelTestRunner {
 
    @Test
    public void testAllFeaturesInParallel() {
        // Run all feature files in parallel with 5 threads
        Results results = Runner.path("classpath:features")
                                .outputCucumberJson(true)
                                .parallel(5);
 
        generateCucumberReport(results.getReportDir());
        assertEquals(0, results.getFailCount(), "Karate Test Suite Failures Detected!");
    }
 
    public static void generateCucumberReport(String karateOutputPath) {
        Collection<File> jsonFiles = FileUtils.listFiles(new File(karateOutputPath), new String[]{"json"}, true);
        List<String> jsonPaths = new ArrayList<>(jsonFiles.size());
        jsonFiles.forEach(file -> jsonPaths.add(file.getAbsolutePath()));
 
        Configuration config = new Configuration(new File("target"), "MyCodeYatra API Suite");
        ReportBuilder reportBuilder = new ReportBuilder(jsonPaths, config);
        reportBuilder.generateReports();
    }
}
```

---

## Key Takeaways

1. **Set Optimal Thread Count**: Match thread counts to host CPU cores (e.g., 5-10 threads).
2. **Tag Filtering**: Execute specific tags in parallel (e.g., `.tags("@smoke")`).
