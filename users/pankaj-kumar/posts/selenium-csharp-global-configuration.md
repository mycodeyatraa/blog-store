---
title: "Global Configuration Manager: Managing Test Data with appsettings.json in Selenium C#"
date: "21-Jan-2025"
description: "Learn how to build a Global Configuration Manager in C# using Microsoft.Extensions.Configuration to dynamically read environment variables and URLs."
categories: ["Selenium C#", "Test Automation"]
tags: ["Selenium", "C#", ".NET", "Configuration", "appsettings.json", "Framework Design"]
author: "Pankaj Kumar"
lastUpdated: "21-Jan-2025"
---

Welcome back to the **Selenium C# Mastery Series**!

In our last blog, we built a highly robust WebDriver Factory. However, if you look closely at our test setup, there is still a major problem:

```cs
driver = BrowserFactory.InitDriver(BrowserType.Edge);
driver.Navigate().GoToUrl("https://practice.mycodeyatra.com/");
```

We are still **hardcoding** the Browser Type and the Base URL directly into our test class! If you want to switch the environment from QA to Staging, or change the browser to Chrome for the CI/CD pipeline, you would have to modify the C# code and recompile the project.

Today, we will fix this by introducing a **Global Configuration Manager** using an `appsettings.json` file.

---

## 1. Installing Required Packages

To parse JSON configuration files easily, Microsoft provides excellent official libraries. Open your terminal and install these two NuGet packages into your project:

```sh
dotnet add package Microsoft.Extensions.Configuration
dotnet add package Microsoft.Extensions.Configuration.Json
```

---

## 2. Creating `appsettings.json`

Create a new file named `appsettings.json` in the root of your project. This file will hold all our global framework variables.

```json
{
  "Environment": "QA",
  "Browser": "Chrome",
  "BaseUrl": "https://practice.mycodeyatra.com/",
  "ImplicitWait": 10
}
```

**Crucial Step:** You must tell Visual Studio (or your build system) to copy this file to the output directory so the test runner can find it.
If you are using a `.csproj` file, ensure it contains this block:

```xml
<ItemGroup>
  <None Update="appsettings.json">
    <CopyToOutputDirectory>PreserveNewest</CopyToOutputDirectory>
  </None>
</ItemGroup>
```

---

## 3. Building the `ConfigReader`

Let's create a new class inside our `Core` folder called `ConfigReader.cs`. This class will use the `ConfigurationBuilder` to load the JSON file and expose the values as properties.

```cs
using Microsoft.Extensions.Configuration;
using System;
using System.IO;
namespace mcyt_sel_csharp.Core
{
    public static class ConfigReader
    {
        private static IConfigurationRoot _configuration;
        static ConfigReader()
        {
            var builder = new ConfigurationBuilder()
                .SetBasePath(Directory.GetCurrentDirectory())
                .AddJsonFile("appsettings.json", optional: false, reloadOnChange: true);
            _configuration = builder.Build();
        }
        public static string Browser => _configuration["Browser"] ?? "Chrome";
        public static string BaseUrl => _configuration["BaseUrl"] ?? throw new Exception("BaseUrl is missing in config!");
        public static int ImplicitWait => int.Parse(_configuration["ImplicitWait"] ?? "10");
        public static string Environment => _configuration["Environment"] ?? "QA";
    }
}
```

By making this class `static`, we ensure the configuration file is only read **once** when the test execution starts, keeping our framework highly performant.

---

## 4. Refactoring the Factory and Test Setup

First, let's parse the string `"Chrome"` from our config file into the `BrowserType` enum we created in the last blog. We can create a quick helper in our `BrowserFactory`:

```cs
public static BrowserType GetBrowserType(string browserName)
{
    if (Enum.TryParse(browserName, true, out BrowserType type))
    {
        return type;
    }
    return BrowserType.Chrome; // Fallback default
}
```

Now, let's create our brand new test, completely free of hardcoded environment variables!

```cs
using NUnit.Framework;
using OpenQA.Selenium;
using mcyt_sel_csharp.Core;
using System;
namespace mcyt_sel_csharp
{
    [TestFixture]
    public class Blog20_GlobalConfiguration
    {
        private IWebDriver driver;
        [SetUp]
        public void Setup()
        {
            // Read from ConfigReader! No hardcoding!
            BrowserType type = BrowserFactory.GetBrowserType(ConfigReader.Browser);
            driver = BrowserFactory.InitDriver(type);
            driver.Manage().Timeouts().ImplicitWait = TimeSpan.FromSeconds(ConfigReader.ImplicitWait);
        }
        [Test]
        public void TestWithGlobalConfig()
        {
            Console.WriteLine($"Running test in {ConfigReader.Environment} environment");
            // Read URL from ConfigReader!
            driver.Navigate().GoToUrl(ConfigReader.BaseUrl);
            Assert.That(driver.Url, Does.Contain("practice"));
        }
        [TearDown]
        public void TearDown()
        {
            if (driver != null)
            {
                driver.Quit();
                driver.Dispose();
            }
        }
    }
}
```

---

## Conclusion

By introducing `appsettings.json` and a `ConfigReader`, you have completely decoupled your test logic from your environment data! 

Now, if DevOps asks you to run the tests in the "Staging" environment on "Firefox", you simply change two strings in your `appsettings.json` file. Zero C# code changes required! This is a massive step toward CI/CD integration.

In the next blog, we will combine everything we've learned to build an elegant **BaseTest** class, which will automatically handle our Setup and Teardown methods behind the scenes!

Happy Automating!
