---
title: Baseline Management: Storing Visual Assets at Enterprise Scale
date: 05-Apr-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [python, selenium, visual-testing, aws-s3, boto3, baselines]
category: Visual Testing
categories: [Visual Testing, Python, UI Automation]
excerpt: >-
  Stop committing thousands of baseline images to Git! Learn how to use Python and the boto3 library to dynamically download visual testing assets from AWS S3 at runtime, keeping your repository lightweight.
readTime: 6 min read
---

# Baseline Management: Storing Visual Assets at Enterprise Scale

In our previous article, we successfully implemented pixel-to-pixel visual testing. But we immediately introduced a massive architectural problem.

Every UI state we want to verify requires a "Baseline" `.png` image. If we have 500 test cases, testing 3 different screen sizes across 3 different browsers, we will generate **4,500 baseline images**. 

If you commit 4,500 `.png` files to your Git repository, your repository size will bloat by gigabytes. Your DevOps team will be furious because `git clone` will take 10 minutes to complete in the CI/CD pipeline!

In this article, we will solve this by implementing **Cloud Baseline Management**. We will keep our Git repository lightweight by dynamically fetching our baseline assets from AWS S3 at runtime!

---

## 1. The Architecture of Cloud Baselines

Instead of storing images in `tests/visual_baselines/`, we will use an AWS S3 Bucket: `s3://mycodeyatra-visual-baselines/`.

**The Workflow:**
1. The Pytest script starts.
2. Before the test runs, Python uses the `boto3` library to download the specific `baseline_homepage_desktop.png` from S3 directly into a temporary OS folder.
3. Selenium captures the current screenshot.
4. The `pixelmatch` comparison runs.
5. The temporary downloaded images are deleted, keeping the testing container perfectly clean!

---

## 2. Managing Baselines with Boto3

First, ensure you have the AWS SDK installed:

```bash
pip install boto3
```

Let's write a utility class that handles downloading (and updating) our baselines in S3.

**utils/visual_asset_manager.py**

```python
import os
import boto3
from botocore.exceptions import ClientError
class VisualAssetManager:
    def __init__(self, bucket_name="mycodeyatra-visual-baselines"):
        self.bucket_name = bucket_name
        self.s3_client = boto3.client('s3', region_name='us-east-1')
        self.temp_dir = "temp_baselines"
        # Ensure temporary directory exists
        os.makedirs(self.temp_dir, exist_ok=True)
    def fetch_baseline(self, image_name: str) -> str:
        """Downloads the baseline image from S3 to a temporary local folder."""
        local_path = os.path.join(self.temp_dir, image_name)
        try:
            print(f"\n[S3] Downloading baseline: {image_name}...")
            self.s3_client.download_file(self.bucket_name, image_name, local_path)
            return local_path
        except ClientError as e:
            # If the baseline doesn't exist, this is a new test! We need to approve it.
            if e.response['Error']['Code'] == "404":
                raise FileNotFoundError(f"Baseline {image_name} not found in S3. Needs approval!")
            raise
    def update_baseline(self, local_image_path: str, new_image_name: str):
        """Uploads a new or updated baseline image to S3."""
        print(f"[S3] Uploading updated baseline: {new_image_name}...")
        self.s3_client.upload_file(local_image_path, self.bucket_name, new_image_name)
        print("✅ Baseline successfully updated in the cloud!")
```

---

## 3. Integrating S3 with Pytest

Now, let's update our visual regression test to seamlessly use our `VisualAssetManager`. 

Notice how we use a standard `try/except` block to gracefully handle brand-new tests where a baseline hasn't been uploaded yet!

**tests/test_cloud_visuals.py**

```python
import os
from selenium import webdriver
from PIL import Image
from pixelmatch.contrib.PIL import pixelmatch
from utils.visual_asset_manager import VisualAssetManager
def test_cloud_visual_regression():
    image_name = "homepage_desktop.png"
    asset_manager = VisualAssetManager()
    # 1. Capture Current State
    driver = webdriver.Chrome()
    driver.set_window_size(1280, 800)
    driver.get("https://httpbin.org/")
    current_path = "current_homepage.png"
    driver.save_screenshot(current_path)
    driver.quit()
    # 2. Fetch the Baseline from AWS S3!
    try:
        baseline_path = asset_manager.fetch_baseline(image_name)
    except FileNotFoundError:
        print("\n⚠️ New Visual Test Detected! Uploading current state as the new Golden Baseline...")
        asset_manager.update_baseline(current_path, image_name)
        assert True, "First run completed. Baseline established."
        return
    # 3. Perform the Pixel Comparison
    print("[Visual] Comparing downloaded baseline against current state...")
    img_baseline = Image.open(baseline_path)
    img_current = Image.open(current_path)
    img_diff = Image.new("RGBA", img_baseline.size)
    mismatch = pixelmatch(img_baseline, img_current, img_diff, threshold=0.1)
    # 4. Clean up the downloaded temporary files so we don't bloat the hard drive!
    os.remove(baseline_path)
    os.remove(current_path)
    assert mismatch < 50, f"UI Regression! {mismatch} pixels changed."
    print("✅ Visual Regression Passed! UI perfectly matches Cloud Baseline.")
```

---

## 4. Execution Output

Let's execute this test. If this is the *first* time we are running the test, it will automatically catch the `FileNotFoundError` from S3 and upload the screenshot to establish the new baseline!

```bash
pytest test_cloud_visuals.py -s
```

```text
============================= test session starts ==============================
platform win32 -- Python 3.12.0, pytest-8.0.0
rootdir: C:\Automation\mycodeyatra-visual
collected 1 item
test_cloud_visuals.py 
[S3] Downloading baseline: homepage_desktop.png...
⚠️ New Visual Test Detected! Uploading current state as the new Golden Baseline...
[S3] Uploading updated baseline: homepage_desktop.png...
✅ Baseline successfully updated in the cloud!
.
============================== 1 passed in 5.21s ===============================
```

If we run the test a second time, it will download that baseline from S3 into memory, compare it, and pass!

## Conclusion

Storing binary assets in a Git repository is an amateur mistake that destroys CI/CD performance.
- By using the `boto3` library, we can shift our heavy `.png` assets entirely to AWS S3.
- Our test code dynamically downloads the golden image, compares it, and deletes it. 
- If a UI redesign happens, you simply call the `update_baseline()` function to overwrite the S3 asset with the new design!

While `pixelmatch` and S3 are incredibly powerful and completely free, maintaining the threshold variance algorithms can get tedious. In our next article, we will introduce **Applitools Eyes**, the industry-leading AI platform that replaces hardcoded pixel-math with Machine Learning cognitive vision!
