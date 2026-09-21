# Day 26 – Configure an EC2 Instance as an Nginx Web Server

## Objective 
The objective of this lab was to deploy an Ubuntu-based EC2 instance named nautilus-ec2 and configure it as a web server using Nginx.


# The Problem
The Nautilus DevOps team needed a new web server as part of the initial infrastructure for an upcoming application.

Manually installing and configuring the web server after every EC2 launch would be time-consuming and difficult to reproduce consistently. The server also needed to be accessible from the internet, which required the appropriate network access through its security group.

User Data Script: Configure the instance to run a user data script during its launch. This script should:

Install the Nginx package.
Start the Nginx service.
Security Group: Ensure that the instance allows HTTP traffic on port 80 from the internet.


# Solution
The solution was to use EC2 User Data to automate the Nginx installation and startup during instance initialization, while allowing inbound HTTP traffic on port 80 through the security group.


## Key Concepts
- EC2
- AMI
- User Data
- Nginx
- Security Groups
- HTTP
- Port 80

## Implementation

### 1. Launch EC2 and choose AMI and Instance type

Choose an available Ubuntu AMI. For example, an Ubuntu Server LTS image.

The exact AMI ID can vary because AWS publishes different Ubuntu images over time and across regions. The lab only requires an available Ubuntu AMI.

Instance type: t2.micro

<img width="1174" height="752" alt="image" src="https://github.com/user-attachments/assets/edc2f611-66a2-45e1-8be4-819182e3cc29" />

## Configure the security group:
Nginx listens for normal HTTP traffic on: TCP 80

<img width="1088" height="668" alt="Screenshot 2026-09-21 060312" src="https://github.com/user-attachments/assets/b2dfceed-7f8d-4722-971b-3be6d58346b8" />

# Enter the User Data

<img width="1154" height="475" alt="image" src="https://github.com/user-attachments/assets/282f9ffe-89e3-41ee-b27f-af22587f4c60" />

#!/bin/bash — tells Ubuntu to run the script with Bash.

apt update — refreshes the package information.

apt install -y nginx — installs Nginx automatically; -y accepts the installation prompt.

systemctl start nginx — starts the Nginx web-server service.

# Wait for the instance

<img width="1718" height="254" alt="image" src="https://github.com/user-attachments/assets/51765a33-14a3-472f-b6b8-3f2bb9259e0d" />

# Verify Nginx
Open a browser and enter: http://<PUBLIC-IP>

<img width="1382" height="302" alt="image" src="https://github.com/user-attachments/assets/d414d828-91a3-4063-811e-66daca282100" />

## Real-World Relevance

This type of configuration is commonly used when deploying web servers in AWS.

**EC2** provides the compute infrastructure, while **Nginx** handles incoming HTTP requests. The **security group** controls which network traffic is allowed to reach the server, and **User Data** automates the initial server configuration.

Using User Data is particularly useful when multiple servers need to be configured consistently, because the installation steps can be automated instead of performed manually.

## Key Takeaways

- **EC2** provides virtual server infrastructure in AWS.
- **User Data** can automate configuration tasks during instance launch.
- **Nginx** is a commonly used web server for serving HTTP applications.
- **Security Groups** control network traffic to EC2 instances.
- **Port 80** is the standard port for HTTP traffic.
- A server can be running correctly but still be inaccessible if the required security-group rules are missing.

The main lesson from this lab was that successful web-server deployment requires both **application configuration** and **network configuration** to work together.



