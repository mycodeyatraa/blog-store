---
title: Salvaging the Evidence: Artifacts & Test Reports
date: 22-Jan-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [typescript, selenium, ci-cd, github-actions, artifacts, reports, screenshots]
category: Selenium TypeScript
categories: [Selenium TypeScript, CI/CD]
excerpt: >-
  Prevent your HTML test reports and failure screenshots from being destroyed by ephemeral cloud servers using GitHub Actions Artifacts.
readTime: 4 min read
---

# Salvaging the Evidence: Artifacts & Test Reports

In Phase 8 and Phase 10 of our curriculum, we spent a significant amount of time configuring our frameworks to generate beautiful HTML test reports and capture high-resolution screenshots whenever a test failed.

However, if we run our test suite inside GitHub Actions, those reports are utterly useless. 

Why? Because GitHub Actions uses **Ephemeral Architecture**.

The moment your test suite finishes running, GitHub instantly destroys the Ubuntu Virtual Machine. Every single file on that server, including your HTML report and your failure screenshots, is permanently deleted.

If a test fails in the cloud, you must be able to view the HTML report to understand *why* it failed. We need a way to extract those files from the VM right before it explodes.

We do this using **Artifacts**.

---

## 1. The `upload-artifact` Action

GitHub provides a built-in action specifically designed to rescue files from a dying Virtual Machine. It takes a folder from the Ubuntu server, zips it up, and attaches it as a downloadable `.zip` file directly to the Workflow Summary page in the UI!

Here is how we integrate it into our `selenium-pipeline.yml`:

```yaml
      - name: Run Selenium Tests (Headless)
        run: npm run test:bdd:smoke
      # New Step: Rescue the reports!
      - name: Upload Test Reports
        uses: actions/upload-artifact@v3
        # VERY IMPORTANT: Always run this step, even if tests fail!
        if: always() 
        with:
          name: cucumber-html-report
          path: reports/ # The folder containing your generated HTML files
          retention-days: 14 # Keep it for 2 weeks to save storage
```

---

## 2. The `if: always()` Clause

Did you notice the `if: always()` condition? This is absolutely critical.

By default, if a step in a GitHub Actions workflow fails (e.g., if `npm run test:bdd:smoke` fails because a Selenium test found a bug), GitHub instantly cancels all remaining steps in the job.

If we don't include `if: always()`, our Artifact upload step will never run when a test fails! And test failures are exactly when we need the HTML report the most! 

By adding `if: always()`, we guarantee that GitHub will execute the `upload-artifact` step regardless of whether the Selenium suite passed or failed.

---

## 3. Uploading Screenshots on Failure

Sometimes, you might save your failure screenshots in a completely different directory than your HTML reports. You can upload multiple artifacts in a single workflow.

In fact, you can use the `if: failure()` clause to only upload screenshots if something actually broke:

```yaml
      - name: Upload Failure Screenshots
        uses: actions/upload-artifact@v3
        if: failure() # Only runs if the previous test step failed
        with:
          name: failure-screenshots
          path: screenshots/
```

---

## 4. Downloading the Artifact

Once the workflow completes, navigate to the **Actions** tab in your GitHub repository. 

Click on the specific workflow run. At the very bottom of the Summary page, you will see a section titled **Artifacts**. 

You can click on `cucumber-html-report` to download the zip file to your local machine. Simply unzip it and double-click `index.html` to view the full, beautiful report of your cloud execution!

## Conclusion

Artifacts are the final piece of the core CI/CD puzzle. You now have a pipeline that automatically triggers, securely injects hidden passwords, executes cross-browser tests in parallel, and safely extracts the HTML evidence before shutting down.

But how do we know if a developer broke something if nobody is actively watching the Actions tab?

In our next tutorial, we will explore **Notifications & Scheduling** to ensure the entire engineering team gets an alert the moment a UI Automation test fails in the cloud!
