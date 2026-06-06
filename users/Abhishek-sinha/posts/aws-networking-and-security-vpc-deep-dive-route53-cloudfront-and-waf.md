---
title: "AWS Networking & Security: VPC Deep Dive, Route53, CloudFront, and WAF"
excerpt: "A deep dive into AWS networking and security — advanced VPC patterns with Transit Gateway and VPC Endpoints, Route53 routing policies and DNS failover, CloudFront content delivery with OAI, and WAF/Shield application protection."
date: 31-May-2026
author: Abhishek-sinha
tags: ["AWS", "VPC", "Route53", "CloudFront", "WAF", "networking", "security", "cloud-computing", "devops"]
---

# AWS Networking &amp; Security: VPC Deep Dive, Route53, CloudFront, and WAF

> **Key insight:** Your network architecture determines your security posture. Every service, every connection, every DNS entry is an attack surface &mdash; design your network and security layers together, not separately.

In Article 1, we introduced VPC fundamentals &mdash; subnets, route tables, Internet Gateways, NAT Gateways, and Security Groups. Now let&rsquo;s go deeper into advanced VPC patterns, DNS management with Route53, content delivery with CloudFront, and application security with WAF and Shield.

## Advanced VPC Patterns

### VPC Peering and Transit Gateway

When your architecture grows beyond a single VPC, you need connectivity between them. VPC Peering creates a direct network route between two VPCs, but it&rsquo;s not transitive &mdash; if VPC A is peered with VPC B and VPC B is peered with VPC C, A cannot reach C through B.

**AWS Transit Gateway** solves this. It acts as a hub-and-spoke router connecting VPCs, VPNs, and Direct Connect in a star topology:


![diagram_1](https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/Abhishek-sinha/aws-networking-and-security-vpc-deep-dive-route53-cloudfront-and-waf/images/diagram_1.png)


**Use Transit Gateway when:** you have more than 3 VPCs, need VPN/Direct Connect integration, or want centralized route management.

**Use VPC Peering when:** you have exactly 2 VPCs that need simple connectivity.

### VPC Endpoints

VPC Endpoints let you access AWS services privately without traversing the internet. Two types:

- **Gateway Endpoints** &mdash; for S3 and DynamoDB. Added as a route table entry, free of charge.
- **Interface Endpoints** &mdash; for all other AWS services (Lambda, SQS, SNS, etc.). Powered by AWS PrivateLink, charged per hour + per GB processed.

> **Key insight:** Always use Gateway Endpoints for S3 and DynamoDB access from private subnets. They&rsquo;re free, and your traffic never leaves the AWS network. Interface Endpoints for other services are worth the cost when compliance requires all traffic to stay within AWS.

### VPC Flow Logs

Flow Logs capture IP traffic metadata for your VPC, subnet, or ENI. They go to CloudWatch Logs or S3.

**What to look for:**
- **REJECT traffic** &mdash; security group or NACL blocking legitimate traffic (debug connectivity issues)
- **Unexpected ACCEPT traffic** &mdash; a security group is too permissive
- **Traffic to/from unknown IPs** &mdash; potential data exfiltration

### Network ACLs vs Security Groups (Revisited)

| Feature | Security Group | Network ACL |
|---------|---------------|-------------|
| Scope | Instance-level (ENI) | Subnet-level |
| State | Stateful (return traffic auto-allowed) | Stateless (return traffic must be explicitly allowed) |
| Rules | Allow only | Allow and Deny |
| Evaluation | All rules evaluated together | Rules evaluated in order (lowest number first) |
| Use case | Application access control | Broad-stroke network boundaries |

## Route53: DNS and Traffic Management

Amazon Route53 is a highly available and scalable Domain Name System (DNS) service. Beyond simple DNS resolution, it offers traffic routing policies, health checks, and domain registration.

### Routing Policies

| Policy | Description | Use Case |
|--------|-------------|----------|
| Simple | Maps one domain to one resource | Basic A/AAAA/CNAME records |
| Weighted | Distributes traffic across resources by weight | Blue/green deployments, A/B testing |
| Latency | Routes to the region with lowest latency | Global applications |
| Failover | Routes to primary, fails over to secondary | Disaster recovery |
| Geolocation | Routes based on user&rsquo;s geographic location | Content localization, compliance |
| Geoproximity | Routes based on geographic location + bias | Traffic shifting between regions |
| Multi-value | Returns up to 8 healthy records | Load balancing without a load balancer |


![diagram_2](https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/Abhishek-sinha/aws-networking-and-security-vpc-deep-dive-route53-cloudfront-and-waf/images/diagram_2.png)


### Health Checks and DNS Failover

Route53 health checks monitor the status of your endpoints (HTTP/HTTPS/TCP) or other CloudWatch alarms. When a health check fails:
1. Route53 stops routing traffic to the unhealthy endpoint
2. DNS TTL determines how quickly clients stop connecting to the failed endpoint
3. For failover routing, traffic goes to the secondary resource

> **Key insight:** Set DNS TTLs low (60 seconds) for active failover scenarios. High TTLs (300+ seconds) improve caching but delay failover &mdash; every client must wait for TTL expiry before retrying DNS resolution.

## CloudFront: Content Delivery at the Edge

CloudFront is AWS&rsquo;s global content delivery network (CDN) with over 600+ points of presence. It caches content at edge locations, reducing latency for end users.

### Origins and Behaviors

A CloudFront distribution can have multiple origins (S3 bucket, ALB, EC2, custom HTTP server) with different behaviors based on URL path patterns:


![diagram_3](https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/Abhishek-sinha/aws-networking-and-security-vpc-deep-dive-route53-cloudfront-and-waf/images/diagram_3.png)


### CloudFront Security Features

- **Origin Access Identity (OAI)** &mdash; restricts S3 bucket access to CloudFront only. The bucket has a policy allowing only the CloudFront OAI principal.
- **AWS WAF integration** &mdash; attach a WAF web ACL to block SQL injection, XSS, and geoblock requests.
- **Signed URLs and Signed Cookies** &mdash; restrict access to premium content.
- **Field-level encryption** &mdash; encrypt sensitive data at the edge before forwarding to the origin.

### Lambda@Edge and CloudFront Functions

Run code at edge locations in response to CloudFront events (viewer request, origin request, viewer response, origin response):

- **CloudFront Functions** &mdash; lightweight (sub-millisecond), for cache key manipulation, URL rewrites, simple header modifications. 2 MB max, no network access.
- **Lambda@Edge** &mdash; full Lambda functions (Node.js, Python) for complex operations like authentication, A/B testing, dynamic content transformation.

## WAF and Shield: Application Security

### AWS WAF

Web Application Firewall protects your web applications from common exploits. Attached to CloudFront, ALB, API Gateway, or AppSync.

**Rule groups:**
- **AWS Managed Rules** &mdash; pre-configured rules for common threats (SQL injection, XSS, LFI, RFI, size constraints)
- **Rate-based rules** &mdash; block IPs exceeding a request threshold (e.g., 2000 requests in 5 minutes)
- **IP reputation lists** &mdash; AWS Threat Intelligence feeds
- **Custom rules** &mdash; your own string matching, regex, IP sets, geo-matching

### AWS Shield

**Shield Standard** &mdash; free, protects against common DDoS attacks (SYN floods, UDP reflection). Automatically enabled on CloudFront, Route53, and ALB.

**Shield Advanced** &mdash; $3,000/month + data transfer fees. Provides enhanced DDoS detection, cost protection, 24/7 access to the AWS DDoS Response Team, and real-time visibility via CloudWatch metrics.

> **Key insight:** For most applications, Shield Standard + WAF with rate-based rules is sufficient protection. Shield Advanced is for high-value targets &mdash; gaming platforms, financial services, or any application where 5 minutes of downtime costs more than $3,000/month.

## Putting It Together: Secure, Global Architecture


![diagram_4](https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/Abhishek-sinha/aws-networking-and-security-vpc-deep-dive-route53-cloudfront-and-waf/images/diagram_4.png)


> **Final Takeaway:** Network and security are not afterthoughts &mdash; design them into your architecture from the start. Use Transit Gateway for multi-VPC connectivity, VPC Endpoints for private AWS access, CloudFront with OAI for secure content delivery, and WAF with managed rules for application protection. Route53 health checks and failover routing tie it all together with reliable DNS. In the next (and final) article, we&rsquo;ll cover production readiness &mdash; monitoring, cost optimization, CI/CD, and high availability patterns.
