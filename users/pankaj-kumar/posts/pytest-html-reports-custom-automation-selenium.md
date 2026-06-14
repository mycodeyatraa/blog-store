---
title: Lightweight Reporting with pytest-html
date: 16-Jun-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [python, selenium, pytest-html, reporting, automation]
category: Reporting and Observability
categories: [Reporting and Observability, Python, Automation]
excerpt: >-
  Need to share test results without hosting a web server? Learn how to use pytest-html to generate self-contained HTML files and automatically inject Base64 failure screenshots using Pytest hooks!
readTime: 6 min read
---

# Lightweight Reporting with pytest-html

In our previous article, we explored the incredible power of the **Allure** reporting framework. However, Allure has one major drawback: it requires Java. Generating an Allure report involves running an external Java web server to compile the JSON results into an HTML site.

Sometimes, you do not want to set up a web server. Sometimes, you just want to run your Selenium tests locally and instantly get a single, self-contained `report.html` file that you can email to a developer or attach to a Jira ticket.

For this, we use **`pytest-html`**. In this article, we will teach you how to generate, customize, and inject screenshots into standalone HTML reports!

---

## 1. Installing pytest-html

`pytest-html` is an official Pytest plugin. It generates a single HTML file containing your entire test execution summary without any external dependencies.

```bash
pip install pytest-html
```

To generate a report, simply append the `--html` flag to your standard Pytest execution command:

```bash
pytest tests/ --html=reports/test_report.html --self-contained-html
```

*(Note: The `--self-contained-html` flag is crucial! It forces Pytest to embed all CSS styles and images directly inside the HTML file as Base64 strings. This means you can email the file, and it will render perfectly without missing assets!)*

---

## 2. Customizing the Report

By default, `pytest-html` provides a clean, functional interface. However, in an Enterprise environment, you might want to add custom metadata (like the Environment name, the QA Engineer's name, or the specific Application Version) to the top of the report.

You can modify the report's metadata by hooking into Pytest's environment setup in `conftest.py`!

**tests/conftest.py**

```python
import pytest
# Add custom environment metadata to the pytest-html summary table
def pytest_configure(config):
    # Add new metadata
    config._metadata['Project Name'] = 'MyCodeYatra E-Commerce'
    config._metadata['Module'] = 'Checkout flow'
    config._metadata['QA Engineer'] = 'Automation Team'
    # Remove default metadata we don't care about
    if 'Packages' in config._metadata:
        del config._metadata['Packages']
# You can even change the title of the HTML page!
@pytest.hookimpl(tryfirst=True)
def pytest_html_report_title(report):
    report.title = "MyCodeYatra Nightly Execution Report"
```

When you run your tests, the generated HTML will proudly display your custom branding and environmental data right at the top!

---

## 3. Injecting Screenshots into pytest-html

A test report is useless if it doesn't show *why* a test failed. Just like we did with Allure, we need to automatically capture a Selenium screenshot upon failure and inject it directly into the `pytest-html` table.

We achieve this by utilizing the `pytest_runtest_makereport` hook. This hook executes after every single test function finishes. If the test "failed", we grab the WebDriver, take a screenshot, encode it as Base64, and embed it into the HTML report as an `Extra`.

**tests/conftest.py**

```python
import pytest
from selenium import webdriver
@pytest.fixture(scope="function")
def driver(request):
    driver_instance = webdriver.Chrome()
    # Attach driver to the request node so the hook can find it!
    request.node.driver = driver_instance 
    yield driver_instance
    driver_instance.quit()
@pytest.hookimpl(hookwrapper=True)
def pytest_runtest_makereport(item, call):
    # Execute all other hooks to obtain the report object
    outcome = yield
    report = outcome.get_result()
    # We only care about the actual test execution phase (not setup/teardown)
    if report.when == "call" or report.when == "setup":
        # Did the test fail?
        if report.failed:
            # Check if the fixture attached a driver to the node
            driver = getattr(item, "driver", None)
            if driver:
                # 1. Take a screenshot and encode it in Base64
                screenshot = driver.get_screenshot_as_base64()
                # 2. Create the HTML string to embed the image
                html_snippet = f'<div><img src="data:image/png;base64,{screenshot}" alt="screenshot" style="width:600px;height:auto;" onclick="window.open(this.src)" align="right"/></div>'
                # 3. Inject it into the pytest-html report
                extra = getattr(report, "extra", [])
                # Import the pytest-html Extras module
                from pytest_html import extras
                extra.append(extras.html(html_snippet))
                report.extra = extra
```

### How does this work?
1. The test function runs and fails.
2. The `pytest_runtest_makereport` hook fires immediately.
3. It detects the failure and fetches the `driver` instance from the current test context.
4. It calls `get_screenshot_as_base64()`.
5. It wraps the raw Base64 image data inside an `<img>` HTML tag and pushes it into the `pytest-html` `extra` array!

When you open `test_report.html`, you will see a beautiful table. Expanding the failed test row will reveal the exact screenshot of the browser at the moment of failure!

## Conclusion

While Allure provides massive trend analysis for CI/CD pipelines, **`pytest-html`** is the king of portability.
- Use `pip install pytest-html`.
- Run your tests with `--html=report.html --self-contained-html` to generate a single, easily shareable file.
- Use the `pytest_configure` hook to inject custom project metadata.
- Use the `pytest_runtest_makereport` hook to automatically embed Base64 Selenium screenshots directly into the failure logs!

In our next article, we will move beyond static reports and explore how to calculate and display **Dashboard Metrics and Test Analytics** across your entire engineering organization!
