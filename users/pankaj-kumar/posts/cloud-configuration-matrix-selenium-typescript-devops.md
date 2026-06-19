---
title: Managing the Chaos: The Cloud Configuration Matrix
date: 14-Feb-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [typescript, selenium, devops, configuration, matrix, env, dotenv, ci-cd]
category: Selenium TypeScript
categories: [Selenium TypeScript, DevOps]
excerpt: >-
  Tame your DevOps environment variables. Learn how to build a dynamic JSON Configuration Matrix and a TypeScript ConfigReader to seamlessly manage BrowserStack, Sauce Labs, and LambdaTest execution parameters.
readTime: 4 min read
---

# Managing the Chaos: The Cloud Configuration Matrix

Over the course of Phase 14, we have transformed our simple local testing framework into a massive, highly scalable Enterprise juggernaut.

We now support:
1. Local Execution (Direct ChromeDriver)
2. Local Docker Execution (Standalone Selenium Container)
3. Cloud Execution (BrowserStack)
4. Cloud Execution (Sauce Labs)
5. Cloud Execution (LambdaTest)

If you try to hardcode all the URLs, Usernames, Access Keys, and capability dictionaries for all 5 of these environments directly into your `hooks.ts` file, your code will become an unmaintainable nightmare.

To solve this, we must build a **Cloud Configuration Matrix**.

---

## 1. The Secrets Vault (.env)

The first rule of Enterprise DevOps: **Never commit a password to GitHub.**

All Usernames and Access Keys must be stripped out of your TypeScript code and moved into a `.env` file. This file must be added to your `.gitignore`.

```env
# .env
BROWSERSTACK_USERNAME=my_bstack_user
BROWSERSTACK_ACCESS_KEY=abc123xyz
SAUCE_USERNAME=my_sauce_user
SAUCE_ACCESS_KEY=def456uvw
LAMBDATEST_USERNAME=my_lambda_user
LAMBDATEST_ACCESS_KEY=ghi789rst
```

When your framework runs in GitHub Actions, you inject these secrets securely using GitHub Secrets!

---

## 2. The Configuration Matrix (config.json)

Instead of hardcoding capability dictionaries (like `"os": "Windows", "browserName": "Edge"`), we extract all the metadata into a single, centralized JSON file called the **Configuration Matrix**.

```json
// config.json
{
  "environments": {
    "local": {
      "gridUrl": "http://localhost:4444/wd/hub",
      "capabilities": {
        "browserName": "chrome"
      }
    },
    "browserstack_win11_edge": {
      "gridUrl": "https://${BROWSERSTACK_USERNAME}:${BROWSERSTACK_ACCESS_KEY}@hub-cloud.browserstack.com/wd/hub",
      "capabilities": {
        "browserName": "Edge",
        "bstack:options": {
          "os": "Windows",
          "osVersion": "11"
        }
      }
    },
    "saucelabs_mac_safari": {
      "gridUrl": "https://${SAUCE_USERNAME}:${SAUCE_ACCESS_KEY}@ondemand.us-west-1.saucelabs.com:443/wd/hub",
      "capabilities": {
        "browserName": "safari",
        "sauce:options": {
          "os": "macOS 13"
        }
      }
    }
  }
}
```

---

## 3. The Parser (ConfigReader.ts)

Finally, we write a highly intelligent TypeScript class that reads the `config.json`, finds the environment the user requested, and dynamically replaces the `${...}` variables with the actual secrets from the `.env` file!

```typescript
import * as fs from 'fs';
import * as dotenv from 'dotenv';
dotenv.config(); // Load the .env file
export class ConfigReader {
  static getEnvironmentConfig(envName: string) {
    const rawData = fs.readFileSync('config.json', 'utf-8');
    // Replace ${SECRET_NAME} with actual process.env values!
    const parsedData = rawData.replace(/\${(.*?)}/g, (_, varName) => {
      return process.env[varName] || '';
    });
    const config = JSON.parse(parsedData);
    if (!config.environments[envName]) {
      throw new Error(`Environment '${envName}' not found in Matrix!`);
    }
    return config.environments[envName];
  }
}
```

---

## 4. The Final Execution

Now, your execution layer is flawlessly clean.

To run tests on BrowserStack:

```bash
TEST_ENV=browserstack_win11_edge npm run test
```

To run tests on Sauce Labs:

```bash
TEST_ENV=saucelabs_mac_safari npm run test
```

Your `hooks.ts` simply calls the ConfigReader:

```typescript
const envConfig = ConfigReader.getEnvironmentConfig(process.env.TEST_ENV);
const driver = await new Builder()
  .usingServer(envConfig.gridUrl)
  .withCapabilities(envConfig.capabilities)
  .build();
```

## Conclusion of Phase 14

You have mastered the DevOps layer. You can containerize your framework, deploy it to Kubernetes, orchestrate it via GitHub Actions, route it to multiple global Cloud Providers, and manage the complexity using a dynamic JSON Configuration Matrix.

But infrastructure is only half the battle.

If your Page Object Models are poorly written, your tests will still fail, no matter how powerful your Kubernetes cluster is.

Welcome to the ultimate challenge. Welcome to **Phase 15: Design Patterns**. In the final phase of this curriculum, we will dive deep into Advanced Architecture, mastering the Singleton, the Factory, and the Strategy pattern!
