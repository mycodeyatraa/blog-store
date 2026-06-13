---
title: Setting Up a Selenium Python Project from Scratch
date: 07-Nov-2024
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [selenium, python, pytest, venv, setup, selenium-manager]
category: UI Automation
categories: [UI Automation, Selenium, Python]
excerpt: >-
  Learn how to isolate dependencies using Python Virtual Environments, install pytest via requirements.txt, and leverage the new Native Selenium Manager to eliminate webdriver downloads.
readTime: 5 min read
---

# Setting Up a Selenium Python Project from Scratch

Now that we understand why Python is an incredible choice for automation engineering, it is time to get our hands dirty.

In this tutorial, we will set up our local development environment, isolate our dependencies using a Virtual Environment, install Selenium and PyTest, and write a script to launch the Chrome browser.

---

## 1. Prerequisites: Python & PIP

Before starting, you must have Python installed on your machine. You can download it from [Python.org](https://python.org). Make sure to check the box that says **"Add python.exe to PATH"** during installation!

To verify your installation, open your terminal and run:
```bash
python --version
pip --version
```
*(Note: If you are on a Mac, you may need to use `python3` and `pip3` depending on your setup).*

---

## 2. Creating a Virtual Environment (venv)

In Java, Maven or Gradle handles your dependencies. In Python, dependencies are installed globally by default. If you install `selenium==4.0.0` for Project A, and `selenium==4.17.0` for Project B, they will conflict.

To solve this, Python uses **Virtual Environments** (`venv`). A `venv` is an isolated folder that contains its own Python executable and its own `pip` packages.

Open your terminal, navigate to your workspace, and run:

```bash
# Create a folder for your framework
mkdir mycodeyatra-selenium-python
cd mycodeyatra-selenium-python
 
# Create the virtual environment (named 'venv')
python -m venv venv
```

Now, you must **activate** the environment so your terminal knows to use the isolated Python instance:

**Windows:**
```bash
venv\Scripts\activate
```

**Mac / Linux:**
```bash
source venv/bin/activate
```
You will know it worked if you see `(venv)` appearing at the beginning of your terminal prompt!

---

## 3. Installing Dependencies

Instead of a `pom.xml`, Python uses a simple text file called `requirements.txt` to track dependencies. 

Create a file named `requirements.txt` in your root folder and add the following:

```text
selenium==4.17.2
pytest==8.0.0
pytest-html==4.1.1
```

Now, install them into your virtual environment:
```bash
pip install -r requirements.txt
```

---

## 4. The Magic of Native Selenium Manager

If you have written Selenium code in the past, you probably remember downloading `chromedriver.exe` manually, putting it in your `C:\` drive, and setting `System.setProperty()`. 

Or, perhaps you used a third-party library like `webdriver-manager` to handle it for you.

**Forget all of that.**

Starting from Selenium v4.6.0, the Selenium team introduced the **Selenium Manager**. It is a tool baked natively into the library. If Selenium detects that you do not have a driver executable in your system PATH, it will automatically reach out to the internet, download the exact driver matching your browser version, and configure it entirely in the background!

Let's test this magic.

---

## 5. Writing the First Script

Create a new file called `test_setup.py` and write the following code:

```python
import time
from selenium import webdriver
 
def test_launch_browser():
    # 1. Initialize the WebDriver (Selenium Manager handles the driver binary automatically!)
    print("\nLaunching Chrome Browser...")
    driver = webdriver.Chrome()
 
    # 2. Maximize the window
    driver.maximize_window()
 
    # 3. Navigate to our practice site
    driver.get("https://mycodeyatra.com")
 
    # 4. Grab the page title
    title = driver.title
    print(f"Page Title is: {title}")
 
    # 5. Sleep for 3 seconds just to visually see it (Never use sleep in real tests!)
    time.sleep(3)
 
    # 6. Quit the browser and kill the process
    driver.quit()
```

Run the script using our newly installed `pytest` runner:
```bash
pytest -s test_setup.py
```
*(Note: The `-s` flag tells pytest to print our console output instead of capturing it).*

You should see Chrome launch, navigate to MyCodeYatra, print the title to the terminal, and close itself!

## Conclusion

You now have a clean, isolated Python environment equipped with the latest versions of Selenium and PyTest. You have also experienced the incredible convenience of the native Selenium Manager.

However, simply launching a browser is only the beginning. In our next article, we will dive deep into the **Selenium WebDriver Architecture**, exploring how Python objects are translated into native OS events via the W3C Protocol!
