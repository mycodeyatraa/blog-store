---
title: Unlimited Power: Kubernetes (K8s) Scaling
date: 10-Feb-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [typescript, selenium, devops, kubernetes, k8s, grid, scaling, docker]
category: Selenium TypeScript
categories: [Selenium TypeScript, DevOps]
excerpt: >-
  Scale your automation to infinity. Learn how to decouple your Selenium Grid from the CI pipeline, deploy it to a Kubernetes cluster, and instantly spin up 100 parallel Chrome instances.
readTime: 4 min read
---

# Unlimited Power: Kubernetes (K8s) Scaling

Until now, we have been spinning up our Selenium Grid directly inside our CI pipeline (like GitHub Actions) using background `services`. 

This architecture is fantastic for small suites. But what happens when you have 5,000 UI tests? 

If you try to run 5,000 tests across 100 parallel Chrome instances inside a single GitHub Actions VM, the VM will run out of memory and completely crash. A single machine simply cannot handle that much computational load.

To scale beyond the limits of a single machine, we must detach our Selenium Grid from the CI pipeline entirely and move it into a **Kubernetes (K8s)** cluster.

---

## 1. What is Kubernetes?

Kubernetes (often abbreviated as **K8s**) is a container orchestration platform. 

Imagine you have 10 separate physical servers (called "Nodes"). Kubernetes binds these 10 servers together into a single "Cluster." You give Kubernetes a YAML file that says, "I need 100 Chrome containers running right now." 

Kubernetes will automatically look at the CPU and RAM available across the 10 servers and distribute the 100 Chrome containers perfectly across the hardware. If Server #4 catches on fire and dies, Kubernetes will instantly detect it and spin up replacement Chrome containers on Server #5.

---

## 2. Decoupling the Architecture

In an Enterprise K8s setup, your architecture looks like this:

1. **The Execution Environment:** GitHub Actions runs your Node.js/Cucumber test executor.
2. **The Selenium Grid Environment:** An external Kubernetes cluster runs 1 Selenium Hub Pod and 100 Selenium Node Pods.

To make this work, the GitHub Actions test executor simply points its `GRID_URL` variable to the public IP address or internal DNS of the Kubernetes cluster:

```bash
docker run -e GRID_URL=http://k8s-selenium.internal.company.com:4444/wd/hub my-selenium-tests
```

---

## 3. Deploying the Grid to Kubernetes

To deploy the Grid, you must write K8s manifests (YAML files). 

First, we define a Deployment for the Hub:

```yaml
# hub-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: selenium-hub
spec:
  replicas: 1
  selector:
    matchLabels:
      app: selenium-hub
  template:
    metadata:
      labels:
        app: selenium-hub
    spec:
      containers:
        - name: selenium-hub
          image: selenium/hub:latest
          ports:
            - containerPort: 4444
            - containerPort: 4442
            - containerPort: 4443
```

Then, we define a Deployment for the Chrome Nodes:

```yaml
# chrome-node-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: selenium-node-chrome
spec:
  replicas: 10 # Start with 10 Chrome browsers!
  selector:
    matchLabels:
      app: selenium-node-chrome
  template:
    metadata:
      labels:
        app: selenium-node-chrome
    spec:
      containers:
        - name: selenium-node-chrome
          image: selenium/node-chrome:latest
          env:
            # Tell the Chrome nodes how to find the Hub
            - name: SE_EVENT_BUS_HOST
              value: "selenium-hub"
            - name: SE_EVENT_BUS_PUBLISH_PORT
              value: "4442"
            - name: SE_EVENT_BUS_SUBSCRIBE_PORT
              value: "4443"
```

---

## 4. The Magic Command: `kubectl scale`

You apply these files to your cluster using the K8s CLI:

```bash
kubectl apply -f hub-deployment.yaml
kubectl apply -f chrome-node-deployment.yaml
```

Now for the magic. 

Let's say it's 2:00 AM, and the massive Nightly Regression suite is about to start. 10 Chrome browsers aren't enough. You need 100. 

You simply execute this single command:

```bash
kubectl scale deployment selenium-node-chrome --replicas=100
```
Within seconds, Kubernetes pulls the Chrome Docker image and distributes 90 new containers across your massive server farm. Your test execution time instantly drops from 3 hours to 5 minutes!

When the test finishes, you scale it back down:

```bash
kubectl scale deployment selenium-node-chrome --replicas=2
```
This is called **Elastic Scaling**, and it saves your company thousands of dollars in cloud infrastructure costs!

## Conclusion

Kubernetes is the ultimate endgame for on-premise, highly scalable test automation. By decoupling the Grid from the CI pipeline, you unlock unlimited scaling potential.

But managing a Kubernetes cluster is a full-time job. You have to patch Linux servers, upgrade Docker versions, and deal with networking configurations.

What if you don't want to manage *any* servers? What if you just want to pay someone else to host the 100 Chrome browsers for you?

In our next tutorial, we will explore the world of Cloud Providers, starting with **BrowserStack TypeScript** integrations!
