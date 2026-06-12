---
title: Multi-User Testing: Admin vs Customer Workflows
date: 28-Aug-2024
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [selenium, java, multi-user-testing, authentication, webdriver, framework]
category: UI Automation
categories: [UI Automation, Selenium, Java]
excerpt: >-
  Learn how to orchestrate multi-user workflows in Selenium by managing multiple isolated WebDriver instances to simulate real-time collaboration between Customers and Admins.
readTime: 5 min read
---

# Multi-User Testing: Admin vs Customer Workflows

In standard UI automation, a test usually executes from the perspective of a single user. But what happens when you need to test a collaborative workflow? 

Imagine a common e-commerce scenario:
1. **User A (Customer)** adds items to a cart and submits an order.
2. **User B (Admin)** logs into the backend portal and approves the order.
3. **User A (Customer)** verifies the order status changed to "Approved".

You cannot perform this sequentially in the same browser window because logging out of the Customer account to log into the Admin account destroys the session state, breaking the real-time flow. In this article, we explore how to run **Multi-User Tests** by managing isolated WebDriver instances simultaneously.

---

## 1. The Challenge of Shared State

If you try to log into two different accounts in two different tabs of the same Chrome instance, they will share the exact same Cookie Jar. Logging into the Admin account in Tab 2 will instantly overwrite the Customer's authentication token from Tab 1. 

To solve this, we must instantiate two completely independent `WebDriver` objects. Because each `ChromeDriver` launches a physically separate browser process with its own temporary user data directory, their cookies and session states remain entirely isolated.

---

## 2. Managing Multiple WebDrivers in One Test

Let's look at how to structure a test that coordinates two different user personas. 

Notice that we initialize two separate `ChromeDriver` instances, giving us a `customerDriver` and an `adminDriver`.

```java
package com.mycodeyatra.tests;
 
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.testng.Assert;
import org.testng.annotations.AfterMethod;
import org.testng.annotations.BeforeMethod;
import org.testng.annotations.Test;
 
public class CollaborativeOrderTest {
 
    private WebDriver customerDriver;
    private WebDriver adminDriver;
 
    @BeforeMethod
    public void setup() {
        // Launch two completely isolated browsers
        customerDriver = new ChromeDriver();
        adminDriver = new ChromeDriver();
    }
 
    @Test
    public void verifyAdminCanApproveCustomerOrder() {
        // 1. Customer logs in and creates an order
        CustomerPortalPage customerPortal = new CustomerPortalPage(customerDriver);
        customerPortal.login("john_doe", "password123");
        String orderId = customerPortal.placeOrder("Laptop");
 
        Assert.assertEquals(customerPortal.getOrderStatus(orderId), "PENDING");
 
        // 2. Admin logs into the backend in their own isolated browser
        AdminDashboardPage adminDashboard = new AdminDashboardPage(adminDriver);
        adminDashboard.login("super_admin", "adminPass!");
        adminDashboard.navigateToOrders();
 
        // Admin approves the specific order created by the customer
        adminDashboard.approveOrder(orderId);
 
        // 3. Customer checks their portal in real-time
        customerDriver.navigate().refresh();
        Assert.assertEquals(customerPortal.getOrderStatus(orderId), "APPROVED", 
                "Customer UI did not reflect Admin approval!");
    }
 
    @AfterMethod
    public void teardown() {
        if (customerDriver != null) customerDriver.quit();
        if (adminDriver != null) adminDriver.quit();
    }
}
```

---

## 3. Alternative: Using Incognito Mode

If launching two entirely separate physical browsers is too resource-heavy for your CI pipeline, an alternative is to launch one normal window and one **Incognito/Private** window using the same WebDriver binary, or simply leveraging Chrome Options. 

While an Incognito window doesn't share cookies with the main session, Selenium's default architecture makes it slightly tricky to drive both *simultaneously* from a single `WebDriver` object without using complex DevTools Protocol (CDP) commands. 

For 99% of enterprise frameworks, creating two `WebDriver` instances is the cleanest, most thread-safe approach.

---

## System Architecture

Here is how the session isolation looks during test execution:

![diagram_1](https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/posts/multi-user-testing-admin-vs-customer-workflows/images/diagram_1.png)

## Considerations for Framework Architecture

If you are using the `ThreadLocal` Driver Factory we built in Phase 4, you must be careful. By design, a `ThreadLocal` map stores exactly *one* WebDriver per execution thread. 

To support multi-user testing in a ThreadLocal architecture, you must either:
1. Temporarily bypass the factory and instantiate raw drivers just for this specific test.
2. Refactor the `DriverFactory` to use a `ThreadLocal<Map<String, WebDriver>>`, allowing a single thread to hold a map of named drivers (e.g., "customer", "admin").

## Conclusion

Multi-user testing bridges the gap between single-user component checks and true end-to-end integration workflows. By managing multiple drivers, you can simulate highly complex, real-time collaboration scenarios just like real users.

In our next article, we will elevate our authentication skills to the cloud by tackling **SSO Authentication**, exploring how to automate logins through complex providers like Azure AD and Okta!
