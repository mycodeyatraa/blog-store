---
title: "Switching Windows and Tabs in Selenium C#"
date: "09-Jan-2025"
description: "Master multi-window automation in Selenium C#. Learn how to handle multiple browser tabs, navigate between them using WindowHandles, and safely close active tabs."
categories: ["Selenium C#", "Test Automation"]
tags: ["Selenium", "C#", ".NET", "Windows", "Tabs", "SwitchTo"]
author: "Pankaj Kumar"
lastUpdated: "09-Jan-2025"
---

Welcome back to the **Selenium C# Mastery Series**!

In modern web applications, it is incredibly common for a link (especially social media links or external resources) to open in a completely new browser tab or window. 

When this happens, Selenium does **not** automatically follow your eyes to the new tab. Its focus remains strictly on the original window. If you try to interact with elements on the new tab without explicitly telling Selenium to switch, your script will fail!

Today, we learn how to master multi-tab automation using `WindowHandles`.

---

### The Anatomy of a Window Handle

Every single tab or window opened by a WebDriver instance is assigned a unique, alphanumeric ID called a **Window Handle**. 

To manage windows, we use three core commands:

1. **`driver.CurrentWindowHandle`**: Returns the ID of the window Selenium is currently focused on.
2. **`driver.WindowHandles`**: Returns a list (specifically, a `ReadOnlyCollection<string>`) of *all* open window IDs.
3. **`driver.SwitchTo().Window(handle)`**: Shifts Selenium's focus to the specified window ID.

### The Standard Workflow

The standard protocol for switching to a new tab looks like this:
1. Save the original window ID.
2. Perform the action that opens the new tab.
3. Loop through all open window IDs.
4. If the ID is *not* the original window ID, switch to it!

---

### Writing the C# Script

Let's put this into action! We will create a new class called `Blog8_Windows`. We will navigate to our sandbox, artificially spawn a new tab pointing to the MyCodeYatra homepage, switch to it, read the title, and then close it to return home!

```csharp
using System;
using OpenQA.Selenium;
using OpenQA.Selenium.Chrome;
using System.Threading;
using System.Collections.ObjectModel;
namespace SeleniumCSharpTutorial
{
    public class Blog8_Windows
    {
        public static void Run()
        {
            Console.WriteLine("Initializing ChromeDriver for Window Switching...");
            ChromeOptions options = new ChromeOptions();
            options.AddArgument("--headless");
            options.AddArgument("--disable-gpu");
            options.AddArgument("--window-size=1920,1080");
            using (IWebDriver driver = new ChromeDriver(options))
            {
                driver.Navigate().GoToUrl("https://practice.mycodeyatra.com/#/form-practice");
                Console.WriteLine("Navigated to Primary Window.");
                Thread.Sleep(2000); 
                // 1. Get the current (original) Window Handle
                string originalWindow = driver.CurrentWindowHandle;
                Console.WriteLine("Original Window Handle: " + originalWindow);
                Console.WriteLine("Original Window Title: " + driver.Title);
                // 2. Open a new Tab using JavaScript
                Console.WriteLine("\nOpening a new tab...");
                IJavaScriptExecutor js = (IJavaScriptExecutor)driver;
                js.ExecuteScript("window.open('https://mycodeyatra.com/', '_blank');");
                Thread.Sleep(3000); // Wait for the new tab to open and load
                // 3. Get all Window Handles
                ReadOnlyCollection<string> allWindows = driver.WindowHandles;
                Console.WriteLine("Total Windows open: " + allWindows.Count);
                // 4. Switch to the new window
                foreach (string window in allWindows)
                {
                    if (window != originalWindow)
                    {
                        Console.WriteLine("Switching to the New Window...");
                        driver.SwitchTo().Window(window);
                        break;
                    }
                }
                // Verify we switched successfully
                Console.WriteLine("New Window Title: " + driver.Title);
                // 5. Close the new window and switch back
                Console.WriteLine("\nClosing the new window...");
                driver.Close(); // Closes ONLY the current active window
                Console.WriteLine("Switching back to Original Window...");
                driver.SwitchTo().Window(originalWindow);
                // Verify we are back
                Console.WriteLine("Active Window Title after switching back: " + driver.Title);
                Console.WriteLine("\nWindows and Tabs script executed successfully!");
            }
        }
    }
}
```

> **💡 Crucial Detail:** Look at step 5! We used `driver.Close()` instead of `driver.Quit()`. `Close()` shuts down the *current active tab*, whereas `Quit()` destroys the entire browser session. After closing the new tab, you **MUST** switch back to the original window ID, or Selenium will be left floating in a void and your next command will crash!

---

### Executing the Code

Update your `Program.cs` to call `Blog8_Windows.Run();` and execute the application:

```bash
dotnet run
```

### 🎯 Console Output

```text
Initializing ChromeDriver for Window Switching...
Navigated to Primary Window.
Original Window Handle: E454FB9A3B48541BBA87F1285AD7A2EE
Original Window Title: MyCodeYatra | Test Automation Sandbox
 
Opening a new tab...
Total Windows open: 2
Switching to the New Window...
New Window Title: MyCodeYatra | The Premium Developer Blogging Network
 
Closing the new window...
Switching back to Original Window...
Active Window Title after switching back: MyCodeYatra | Test Automation Sandbox
 
Windows and Tabs script executed successfully!
```

### Summary

You did it! You have officially conquered Phase 3. You now possess the power to confidently navigate complex applications involving multiple browser windows and external tabs without losing your WebDriver session.

This concludes the foundational interaction techniques. Stay tuned for the next phase of our journey!
