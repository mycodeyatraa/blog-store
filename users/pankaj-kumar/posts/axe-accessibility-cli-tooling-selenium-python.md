---
title: Shifting Left: Accessibility CLI Tooling in Python
date: 26-Feb-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [python, selenium, accessibility, cli, devops, shift-left, pre-commit]
category: Selenium Python
categories: [Selenium Python, Accessibility]
excerpt: >-
  Why wait for E2E tests to fail? Shift accessibility testing left by integrating the Axe CLI directly into your Python pre-commit hooks and local DevOps pipelines.
readTime: 4 min read
---

# Shifting Left: Accessibility CLI Tooling in Python

In our previous tutorials, we integrated Axe-Core natively into our Selenium WebDriver to test accessibility during complex UI flows. 

However, running a full Selenium suite is slow. If a frontend developer introduces an accessibility violation (like a missing `alt` tag on a newly added image), they shouldn't have to wait 30 minutes for the end-to-end suite to fail to discover their mistake.

We need to **Shift Left**. We need to empower developers to run accessibility checks instantly from their terminal, before they even commit their code. 

In this tutorial, we will explore **Accessibility CLI Tooling** and learn how to run Axe checks headlessly via the command line!

---

## 1. What is the `@axe-core/cli`?

Deque Systems (the creators of Axe) maintains an official Command Line Interface. While it is built on Node.js, it can be easily integrated into any Python-centric DevOps pipeline or Makefile.

The CLI spins up a headless Chromium browser, navigates to a URL, runs the Axe-Core engine, and prints the results directly to `stdout`.

### Installation

Since it is a Node module, you will need `npm` installed. Run the following command globally:

```bash
npm install -g @axe-core/cli
```

Verify the installation:

```bash
axe --version
```

---

## 2. Running Basic CLI Scans

You can run a scan against any local or public URL with a single command:

```bash
axe https://practice.mycodeyatra.com
```

The output will look something like this directly in your terminal:
```
Testing https://practice.mycodeyatra.com...
2 Violations Found:
1. color-contrast: Elements must have sufficient color contrast
   Impact: serious
   Help: https://dequeuniversity.com/rules/axe/4.4/color-contrast
   Nodes: 1
2. image-alt: Images must have alternate text
   Impact: critical
   Help: https://dequeuniversity.com/rules/axe/4.4/image-alt
   Nodes: 2
```

If any violations are found, the process exits with a non-zero exit code (`exit 1`), which makes it perfect for blocking a CI/CD pipeline!

---

## 3. Integrating Axe CLI with Python Subprocess

If you are orchestrating a complex local testing environment in Python, you might want to wrap the CLI execution in a Python script so you can trigger it programmatically (e.g., as part of a pre-commit hook).

You can use Python's built-in `subprocess` module to execute the CLI:

```python
import subprocess
import sys
def run_axe_cli(url: str):
    """
    Executes the axe-core CLI against a given URL.
    """
    print(f"Running Axe CLI against {url}...")
    # We use --exit to ensure it returns an exit code of 1 if violations exist
    command = ["axe", url, "--exit"]
    try:
        # Run the command and capture output
        result = subprocess.run(
            command,
            capture_output=True,
            text=True,
            check=False # We don't want it to throw a Python exception on fail
        )
        # Print the stdout from Axe
        print(result.stdout)
        if result.returncode != 0:
            print("❌ Accessibility checks failed!")
            sys.exit(1)
        else:
            print("✅ Accessibility checks passed!")
            sys.exit(0)
    except FileNotFoundError:
        print("Error: '@axe-core/cli' is not installed. Please run 'npm install -g @axe-core/cli'")
        sys.exit(1)
if __name__ == "__main__":
    run_axe_cli("https://practice.mycodeyatra.com")
```

---

## 4. Bypassing Authentication via CLI

A common challenge with CLI tools is scanning authenticated pages (like a dashboard). By default, `axe` will just scan your login page.

The Axe CLI allows you to pass custom headers or cookies! You can use your Python framework to programmatically grab an auth token, then pass it to the CLI:

```bash
# Passing a Bearer token
axe https://practice.mycodeyatra.com/dashboard --header "Authorization: Bearer eyJhbGciOi..."
```

Or pass a session cookie:

```bash
# Passing a Cookie string
axe https://practice.mycodeyatra.com/dashboard --header "Cookie: session_id=abc123xyz"
```

---

## 5. Adding Pre-Commit Hooks

The ultimate way to "Shift Left" is to enforce this script locally before a developer is even allowed to type `git commit`.

Using the Python `pre-commit` framework, you can add a local hook that runs the Axe CLI against the local dev server (`http://localhost:3000`):

Create a `.pre-commit-config.yaml`:

```yaml
repos:
  - repo: local
    hooks:
      - id: axe-accessibility-check
        name: Accessibility CLI Check
        entry: python scripts/run_axe_cli.py http://localhost:3000
        language: system
        pass_filenames: false
```

Now, every time a developer tries to commit new frontend code, the Axe CLI will silently boot up, scan the local server, and reject the commit if they introduced a WCAG violation!

## Conclusion

Combining standard Selenium tests (for dynamic UI state validations) with the Axe CLI (for instant, shift-left feedback) creates an impenetrable accessibility testing strategy. 

In our final Accessibility tutorial, we will zoom out and discuss how to implement a complete **Enterprise Accessibility Strategy**, including team culture, risk matrices, and VPATs!
