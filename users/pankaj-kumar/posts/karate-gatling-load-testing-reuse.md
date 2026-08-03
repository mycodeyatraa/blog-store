---
title: Karate Gatling: Reusing Functional API Tests for Performance Testing
date: 25-Feb-2026
lastUpdated: 25-Feb-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [karate, gatling, performance, load-testing, scalability]
category: API Karate
categories: [API Karate, Performance Testing, Gatling]
excerpt: >-
  Turn functional tests into load tests instantly! Reuse Karate API feature files with Gatling for high-concurrency performance validation.
readTime: 8 min read
---

# Karate Gatling: Reusing Functional API Tests for Performance Testing

Creating separate functional and performance test scripts doubles maintenance overhead. If an API payload format changes, engineers must update both functional test specs and JMeter / Gatling scripts.

**Karate Gatling Integration** eliminates duplicate scripting by allowing engineers to execute existing Karate `.feature` scenarios as load generators under **Gatling** high-concurrency simulations.

---

## 1. How Karate + Gatling Integration Works

Gatling manages virtual user concurrency, ramp-up schedules, and response time metrics, while Karate executes the actual HTTP request payloads, assertions, and payload transformations.

```
 +---------------------+       Ramp Users & Load      +------------------------+
 | Gatling Simulation  | ---------------------------> | Karate Feature File    |
 | (Scala / Java DSL)  | <--------------------------- | (checkout.feature)     |
 +---------------------+       Response Metrics       +------------------------+
```

---

## 2. Defining a Gatling Simulation Class

Write a Gatling Simulation in Java or Scala that executes your Karate feature file:

**src/test/java/com/mycodeyatra/simulations/ApiLoadSimulation.java**

```java
package com.mycodeyatra.simulations;
 
import static com.intuit.karate.gatling.KarateDsl.*;
import static io.gatling.javaapi.core.CoreDsl.*;
 
import io.gatling.javaapi.core.ProtocolBuilder;
import io.gatling.javaapi.core.ScenarioBuilder;
import io.gatling.javaapi.core.Simulation;
 
public class ApiLoadSimulation extends Simulation {
 
    // 1. Create Karate Protocol Definition
    ProtocolBuilder protocol = karateProtocol();
 
    // 2. Wrap existing Karate feature file into a Gatling Scenario
    ScenarioBuilder createOrderScenario = scenario("Create Order Load Test")
        .exec(karateFeature("classpath:features/create-order.feature"));
 
    {
        // 3. Configure Ramp-up load profile
        setUp(
            createOrderScenario.injectOpen(
                nothingFor(4),
                atOnceUsers(10),
                rampUsers(50).during(30)
            )
        ).protocols(protocol);
    }
}
```

---

## 3. Running Gatling Simulations & Reports

Run the simulation via Maven:

```bash
mvn clean test-compile gatling:test -Dgatling.simulationClass=com.mycodeyatra.simulations.ApiLoadSimulation
```

Gatling automatically generates visual HTML performance reports detailing throughput (RPS), mean latency, 95th percentile response times, and failure rates.

---

## Key Best Practices

1. **Keep Load Scenarios Isolated**: Avoid complex setup fixtures inside performance-tested feature files.
2. **Assertions Under Load**: Leverage Karate `match` statements to ensure APIs return correct response bodies even under 500 RPS load.
