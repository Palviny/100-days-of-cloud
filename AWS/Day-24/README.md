# Day 24 — Setting Up an Application Load Balancer for an EC2 Instance

## Objective

Set up an Application Load Balancer (ALB) in front of an existing
Nginx web server running on an EC2 instance.

### Resources

| Resource | Name |
|---|---|
| EC2 | `devops-ec2` |
| Load Balancer | `devops-alb` |
| Target Group | `devops-tg` |
| ALB Security Group | `devops-sg` |
| Protocol | HTTP |
| Port | 80 |
| Region | `us-east-1` |

---

## The Problem

The application was currently running directly on an EC2 instance.
The team wanted a load balancer in front of the server so that incoming web traffic could be routed through a dedicated entry point.

This architecture also provides a foundation for adding additional
EC2 instances later.

---

## Solution

We created an **Application Load Balancer**, a **target group**, and
a dedicated security group.

The traffic flow is:

```text
Internet
   │
   │ HTTP :80
   ▼
devops-alb
   │
   ▼
devops-tg
   │
   ▼
devops-ec2
   │
   ▼
Nginx :80

## Implementation

1. Create the ALB security group

Created:

devops-sg

Inbound rule:

HTTP | TCP | 80 | 0.0.0.0/0

This allows public HTTP traffic to reach the load balancer.

2. Create the target group
Created: devops-tg

Configuration:

Target type: Instances
Protocol: HTTP
Port: 80
Health check: HTTP /

The existing devops-ec2 instance was registered as a target.

A target group tells the ALB which backend resources should receive traffic.

3. Create the Application Load Balancer

Created: devops-alb

### Configuration:

Application Load Balancer
Internet-facing
IPv4
Same VPC as devops-ec2
Security group: devops-sg
Listener: HTTP port 80
Default action: forward to devops-tg

4. Configure EC2 security

The EC2 instance was using the default security group.

The required rule was added:

HTTP | TCP | 80 | Source: devops-sg

This allows HTTP traffic from the ALB to reach Nginx while avoiding the need to expose the backend specifically to the
entire internet.

 ### Verification

The target initially reported as unhealthy because the EC2 security group did not allow HTTP traffic from the ALB.

After adding the appropriate rule: devops-sg → EC2 port 80

the target became: Healthy

Finally, the ALB DNS name was opened in a browser and successfully returned the Nginx welcome page.

This confirmed the complete traffic path was working.

## Real-World Relevance

Application Load Balancers are commonly used to:

Distribute HTTP/HTTPS traffic
Route requests to application servers
Perform health checks
Support highly available architectures
Scale applications across multiple EC2 instances

With multiple backend servers, the ALB can distribute traffic between healthy targets.

### Key Takeaway

An Application Load Balancer acts as a public entry point for HTTP/HTTPS applications.

The listener accepts incoming traffic, the target group defines the backend servers, and health checks determine whether those servers are available to receive requests.

A key troubleshooting lesson from this lab was that both sides of the connection need appropriate security-group rules:

Internet → ALB :80
ALB → EC2 :80

Both must be allowed for the application to work through the
load balancer.
