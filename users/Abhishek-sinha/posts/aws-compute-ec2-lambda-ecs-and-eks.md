---
title: "AWS Compute: EC2, Lambda, ECS, and EKS"
excerpt: "A practical guide to AWS compute services — EC2 Auto Scaling, Lambda serverless functions, ECS Fargate containers, and EKS Kubernetes — with decision framework for choosing the right service."
date: 31-May-2026
author: Abhishek-sinha
tags: ["AWS", "EC2", "Lambda", "ECS", "EKS", "cloud-computing", "containers", "serverless", "devops", "infrastructure"]
---

# AWS Compute: EC2, Lambda, ECS, and EKS

> **Key insight:** Choosing the right compute service is the single most impactful architectural decision in AWS. Each service optimizes for a different tradeoff between control, scalability, and operational overhead.

In Article 1, we covered the four pillars of AWS &mdash; IAM, VPC, EC2, and S3. Now it&rsquo;s time to zoom in on AWS compute. The platform offers four primary compute services, each at a different point on the spectrum from full control to zero operations. Understanding when to use each &mdash; and how to combine them &mdash; separates architects who build maintainable systems from those who create operational nightmares.

## EC2: Maximum Control, Maximum Responsibility

We introduced EC2 in Article 1, but instances in isolation aren&rsquo;t a production architecture. You need Auto Scaling groups for elasticity, launch templates for consistency, and placement groups for performance.

### Auto Scaling Groups

An Auto Scaling Group (ASG) maintains a desired count of EC2 instances across multiple Availability Zones. It automatically replaces unhealthy instances and scales based on metrics:


![diagram_1](https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/Abhishek-sinha/aws-compute-ec2-lambda-ecs-and-eks/images/diagram_1.png)


**Scaling policies:**
- **Target tracking** &mdash; maintain CPU at 50%, request count at 1000, or custom metrics
- **Step scaling** &mdash; add 2 instances when CPU > 70%, add 4 when CPU > 90%
- **Scheduled scaling** &mdash; predictable patterns (scale up at 8 AM, down at 8 PM)

> **Key insight:** Use target tracking for 80% of workloads. It adjusts smoothly to traffic changes without the oscillation that step scaling can cause.

### Launch Templates vs Launch Configurations

Launch templates are the modern replacement for launch configurations. They support versioning (roll back to v3 easily), T2/T3 unlimited credits, and are required for newer features like EC2 Fleets.

### Placement Groups

| Type | Description | Use Case |
|------|-------------|----------|
| Cluster | Instances close together in a single AZ | HPC, low-latency inter-node communication |
| Spread | Instances on distinct hardware (7 per AZ max) | Critical instances that must be isolated |
| Partition | Instances grouped into logical partitions (max 7 per AZ) | Distributed systems like Kafka, Cassandra |

## Lambda: No Servers, Just Code

AWS Lambda executes code in response to events with zero infrastructure management. You upload code, configure a trigger, and AWS runs it &mdash; scaling from zero to thousands of concurrent executions in seconds.

### Execution Model

Lambda functions run in isolated sandboxes with a lifespan of up to 15 minutes (900 seconds). Each invocation gets a fresh runtime environment, though AWS may reuse warm containers for subsequent invocations.

**Cold starts** occur when a new sandbox must be created &mdash; adding 100ms to 2 seconds of latency depending on runtime and configuration. Provisioned Concurrency keeps a specified number of environments warm, eliminating cold starts at a cost.


![diagram_2](https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/Abhishek-sinha/aws-compute-ec2-lambda-ecs-and-eks/images/diagram_2.png)


### Lambda Use Cases

- **API backends** &mdash; combine with API Gateway for serverless REST APIs
- **Event processors** &mdash; react to S3 uploads, DynamoDB streams, SQS messages
- **Scheduled tasks** &mdash; CloudWatch Events trigger Lambda on a cron schedule
- **Infrastructure automation** &mdash; auto-tag resources, enforce compliance rules
- **Data transformations** &mdash; process records from Kinesis or DynamoDB Streams

### Function Configuration

| Setting | Range | Impact |
|---------|-------|--------|
| Memory | 128 MB - 10,240 MB | CPU scales proportionally with memory |
| Timeout | 1 sec - 900 sec (15 min) | Longer = higher cost if code is slow |
| Ephemeral storage | 512 MB - 10,240 MB | /tmp directory for temporary files |
| Concurrency | 0 - 1,000 (default regional limit) | Controls how many run simultaneously |

> **Key insight:** Memory is the only CPU control. Doubling memory from 512 MB to 1 GB often makes code run 2x faster &mdash; and costs the same because the billing is memory &times; duration. Profile your functions and right-size the memory configuration.

## ECS: Containers Without Kubernetes

Amazon Elastic Container Service runs Docker containers on a managed cluster. It comes in two launch types:

**Fargate** &mdash; serverless containers. Specify CPU and memory, AWS manages the infrastructure. No EC2 instances to manage, patch, or scale.

**EC2** &mdash; you manage the cluster of EC2 instances. More control over cost optimization (Reserved Instances, Spot) and custom configurations.

### Task Definitions and Services

A **task definition** is the blueprint for your container &mdash; image, port mappings, environment variables, resource limits, IAM role. A **service** maintains a desired count of tasks behind a load balancer.


![diagram_3](https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/Abhishek-sinha/aws-compute-ec2-lambda-ecs-and-eks/images/diagram_3.png)


### When to Use ECS vs Lambda

| Criterion | Lambda | ECS Fargate | ECS EC2 |
|-----------|--------|-------------|---------|
| Execution duration | < 15 min | Any | Any |
| Memory | Up to 10 GB | Up to 120 GB | Up to 120 GB |
| Cold start tolerance | Yes | No (always warm) | No (always warm) |
| GPU requirement | No | No | Yes |
| Custom networking | VPC | VPC + ENI | VPC + ENI |
| Cost model | Per-invocation | Per-hour (resources) | Per-hour (instance) |

## EKS: Kubernetes on AWS

Amazon Elastic Kubernetes Service manages the Kubernetes control plane. You bring the worker nodes (EC2 or Fargate) and Kubernetes manifests.

### EKS Architecture

EKS runs the Kubernetes control plane across three AZs for high availability. Worker nodes join the cluster via a CNI plugin that assigns VPC IP addresses directly to pods &mdash; meaning pods have the same IP address space as your VPC.

### EKS Node Groups

- **Managed node groups** &mdash; AWS creates and manages the EC2 Auto Scaling groups for worker nodes
- **Self-managed nodes** &mdash; you create and manage the worker nodes with your own AMI
- **Fargate profiles** &mdash; run pods as Fargate tasks, no worker nodes at all (useful for batch jobs and small workloads)

### EKS Key Concepts

- **Pod identity** &mdash; each pod gets an IAM role via IRSA (IAM Roles for Service Accounts), no hardcoded credentials
- **Cluster autoscaler** &mdash; automatically adds/removes worker nodes based on pending pods
- **Horizontal pod autoscaler** &mdash; adjusts pod count based on CPU/memory/custom metrics
- **AWS Load Balancer Controller** &mdash; creates ALBs and NLBs from Kubernetes Ingress resources

> **Key insight:** EKS adds significant operational complexity over ECS &mdash; you manage the control plane updates, worker node AMI updates, CNI plugins, and Kubernetes upgrades. Choose EKS only when you need Kubernetes-specific features (custom scheduling, operator patterns, multi-cloud portability) or when your organization already has Kubernetes expertise.

## Choosing the Right Compute Service

Here&rsquo;s a decision framework:


![diagram_4](https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/Abhishek-sinha/aws-compute-ec2-lambda-ecs-and-eks/images/diagram_4.png)


> **Final Takeaway:** Start with Lambda if your workload fits (short, event-driven, simple). Move to ECS Fargate when you need longer runtimes or more resource control. Choose ECS on EC2 when you need GPUs or cost optimization via Spot/Reserved Instances. Only adopt EKS if you already have Kubernetes expertise or need multi-cloud portability. Each step up the control ladder adds operational cost &mdash; choose the least powerful service that meets your requirements.
