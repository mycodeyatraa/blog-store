---
title: AWS Fundamentals: IAM, VPC, EC2, and S3
date: 2026-05-31
author: Abhishek-sinha
authorName: Abhishek Sinha
authorRole: Full Stack And Mobile Developer
authorAvatar: 
authorBio: Full Stack And Mobile Developer
authorGithub: https://github.com/abhisheksinha20p
authorLinkedin: https://www.linkedin.com/in/abhishek-sinha-0897aa23b/
tags: [aws, fundamentals, iam, ec2, s3]
category: Aws
categories: [Aws, Fundamentals, Iam]
excerpt: >-
  AWS Fundamentals: IAM, VPC, EC2, and S3

> Key insight: Mastering these four AWS services — IAM, VPC, EC2, and S3 — gives you the foundation to build 90% of cloud architectures.. Everything else exten
readTime: 6 min read
---

# AWS Fundamentals: IAM, VPC, EC2, and S3

> **Key insight:** Mastering these four AWS services — IAM, VPC, EC2, and S3 — gives you the foundation to build 90% of cloud architectures. Everything else extends from here.

When developers first approach AWS, the sheer number of services — over 200 at last count — creates a paralysis of choice. Where do you start? The answer is simpler than it seems: four services form the bedrock of every AWS deployment. IAM for security, VPC for networking, EC2 for compute, and S3 for storage. Understanding how these interact is the difference between a clean, maintainable architecture and a security-nightmare spaghetti cloud.

## IAM: The Keys to the Kingdom

Identity and Access Management is the first service you touch in any AWS account, and for good reason. IAM is AWS's centralized authentication and authorization system. Every API call, every console login, every resource operation passes through IAM's decision engine.

### Core Components

**Users** represent individual people or service accounts. Each user has a unique ARN and can have passwords (for the console), access keys (for APIs/CLI), or both. Best practice: never use the root user for daily work.

**Groups** are collections of users. Attach policies to groups, not individuals — a user inherits all permissions from their groups. This scales far better than per-user permissions when your team grows from 3 to 300.

**Roles** are identities you assume, not login as. Roles are the AWS way of doing cross-account access, EC2 instance permissions, and Lambda execution privileges. Unlike users, roles don't have permanent credentials — they issue temporary security tokens via STS (Security Token Service).

**Policies** are JSON documents that define permissions. An explicit deny always overrides an allow. AWS evaluates all policies (identity-based, resource-based, permissions boundaries, service control policies) before making an access decision.


![diagram_1](https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/Abhishek-sinha/aws-fundamentals-iam-vpc-ec2-and-s3/images/diagram_1.png)


> **Key insight:** The principle of least privilege isn't optional in production. Start with a deny-all policy and add only what's needed. Tools like IAM Access Analyzer help identify unused permissions.

## VPC: Your Private Network in the Cloud

A Virtual Private Cloud is your isolated network within AWS. Think of it as your own data center in the cloud — you control IP addressing, subnets, routing, and connectivity.

### Subnets and Route Tables

A VPC spans multiple Availability Zones. Within each AZ, you create subnets — public (with direct internet access via an Internet Gateway) and private (no direct internet access, often with a NAT Gateway for outbound traffic).

**Public subnets** hold resources that need direct internet access: load balancers, bastion hosts, NAT gateways. Their route table has a route to an Internet Gateway (IGW).

**Private subnets** hold everything else: application servers, databases, cache clusters. Their traffic goes through a NAT Gateway for outbound internet (e.g., to download updates) but cannot be reached directly from the internet.


![diagram_2](https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/Abhishek-sinha/aws-fundamentals-iam-vpc-ec2-and-s3/images/diagram_2.png)


**Security Groups** act as instance-level firewalls. They're stateful (return traffic is automatically allowed) and support allow rules only. Each security group is associated with one or more resources.

**Network ACLs** are stateless subnet-level firewalls. They support both allow and deny rules and are evaluated in order. Use NACLs for broad-stroke deny rules (block a known bad IP range) and security groups for granular application access.

> **Key insight:** When something doesn't connect, 90% of the time it's a security group rule missing. Enable VPC Flow Logs from day one — they're invaluable for debugging connectivity issues.

## EC2: The Workhorse of AWS

Elastic Compute Cloud provides virtual servers in the cloud. While serverless is trendy, EC2 remains essential for workloads requiring custom AMIs, GPU instances, or specific hypervisor configurations.

### Instance Types and Families

AWS categorizes EC2 instances into families optimized for different workloads:

| Family | Use Case | Example |
|--------|----------|---------|
| General purpose (t, m) | Balanced compute, memory, networking | t3.medium, m5.large |
| Compute optimized (c) | Batch processing, gaming, HPC | c5.2xlarge |
| Memory optimized (r, x) | In-memory databases, caching | r5.large, x1e.32xlarge |
| Storage optimized (i, d) | Data warehousing, log processing | i3.large, d2.2xlarge |
| GPU (p, g) | ML training, video rendering | p3.2xlarge, g4dn.xlarge |

### Launch Configuration

Every EC2 instance starts from an **Amazon Machine Image (AMI)** — a template with OS, software, and configuration. You can use AWS-provided AMIs (Amazon Linux 2, Ubuntu), AWS Marketplace AMIs, or your own custom AMIs built with tools like Packer.

**User data** scripts run at first boot. Use them for initial configuration — installing packages, pulling application code, registering with a configuration manager.

**Instance metadata** at `http://169.254.169.254/latest/meta-data/` provides runtime information about the instance. This is how instances discover their environment without hardcoded configuration.

### Storage Options

| Volume Type | Max IOPS | Max Throughput | Use Case |
|-------------|----------|----------------|----------|
| gp3 (General Purpose SSD) | 16,000 | 1,000 MB/s | Most workloads, boot volumes |
| io2 (Provisioned IOPS SSD) | 256,000 | 4,000 MB/s | Databases, latency-sensitive apps |
| st1 (Throughput HDD) | 500 | 500 MB/s | Big data, log processing |
| sc1 (Cold HDD) | 250 | 250 MB/s | Infrequent access, lowest cost |

> **Key insight:** gp3 volumes are almost always the right choice. They offer baseline performance without provisioned IOPS costs and can burst for most workloads.

## S3: Infinite, Durable Object Storage

Simple Storage Service is AWS's object storage platform — infinitely scalable, 99.999999999% durability, and the foundation for data lakes, backups, static websites, and content distribution.

### Storage Classes

S3 automatically migrates objects between storage classes based on lifecycle policies, optimizing cost as data ages:


![diagram_3](https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/Abhishek-sinha/aws-fundamentals-iam-vpc-ec2-and-s3/images/diagram_3.png)


### Bucket Policies and Access Control

S3 offers multiple access control mechanisms:

- **Bucket policies** — resource-based JSON policies attached to the bucket
- **IAM policies** — identity-based policies on users/roles
- **ACLs** — legacy access control lists (avoid for new deployments)
- **Pre-signed URLs** — time-limited URLs for temporary access
- **Origin Access Identity (OAI)** — restrict access to CloudFront only

### Versioning and Lifecycle

Enable **versioning** for data protection against accidental deletion. Combined with lifecycle rules, you can:
- Transition older versions to Glacier after 30 days
- Permanently delete noncurrent versions after 365 days
- Expire incomplete multipart uploads after 7 days

> **Key insight:** S3 is not a filesystem. There's no rename operation, list operations are eventually consistent for some regions, and you pay per-request. Design your key naming scheme thoughtfully — S3 scales by partitioning on key prefixes.

## Putting It All Together

Here's a simplified architecture showing how IAM, VPC, EC2, and S3 work together in a typical web application:


![diagram_4](https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/Abhishek-sinha/aws-fundamentals-iam-vpc-ec2-and-s3/images/diagram_4.png)


The EC2 instance assumes an IAM role to read from S3 and write to the database. The load balancer in a public subnet receives traffic, private subnets host the application and database, and S3 serves static assets through CloudFront. Every component is locked down by security groups and IAM policies.

> **Final Takeaway:** IAM, VPC, EC2, and S3 are the four pillars of AWS. Master these, and every other service — Lambda, ECS, DynamoDB, ElastiCache — becomes an extension of the mental model you've already built. Get the fundamentals right: least privilege by default, network isolation in tiers, instance rightsizing, and S3 lifecycle automation.
