# Auto Scaling Group + ALB Lab (Ubuntu)

## Objective
- Launch EC2 instances automatically using **Auto Scaling Group (ASG)**
- Attach **Application Load Balancer (ALB)**
- Simple Nginx web server on each instance
- Test dynamic scaling based on CPU


## 🧩 Architecture
User → ALB (HTTP 80) → Target Group (ASG Instances)

## ⚙️ Configuration Summary

| Component          | Details                        |
|-------------------|--------------------------------|
| OS                 | Ubuntu 22.04                  |
| Web Server         | Nginx                          |
| Instance Type      | t3.micro                       |
| Min/Desired/Max    | 3 / 3 / 10                      |
| Scaling Policy     | Add instance @ CPU > 40%       |
| Monitoring         | CloudWatch                     |

---

## 🪜 Steps

### 1️⃣ Launch Template
User Data script used:
```bash
#!/bin/bash
apt-get update -y
apt-get install nginx -y
echo "This is my $(hostname)" > /var/www/html/index.html
service nginx start


## Pre-requisites
- AWS Account
- VPC with 2 public subnets
- Key Pair for SSH
- Security Groups:
  - `ALB-LB`: HTTP 80 → 0.0.0.0/0
  - `ASG-SG`: HTTP 80 → ALB-SG, SSH 22 → Your IP (optional) 

## Steps

### 1. Create Launch Template
- Name: myfirsttemplate
- AMI: Ubuntu latest
- Instance Type: t3.micro
- Key Pair: EC2-key
- Security Group: ASG-SG
- User Data: 
### 1️⃣ Launch Template
User Data script used:
```bash
#!/bin/bash
apt-get update -y
apt-get install nginx -y
echo "This is my $(hostname)" > /var/www/html/index.html
service nginx start


### 2. Create Auto Scaling Group
- Name: myAUTOSG
- Launch Template: myfirsttemplate
- Min: 3, Desired: 3, Max: 10
- Subnets: select 2 public subnets
- Target Group: create new for ALB

### 3. Create ALB
- Name: LB-ASG
- Subnets: same 2 public subnets
- Security Group: ALB-SG
- Listener: HTTP 80
- Routing → Target Group: your ASG target group

### 4. Test
- Open **ALB DNS** in browser → should see:
This is my <instance-hostname>

##Step 5: Dynamic Scaling Test

SSH to one of the instances
Sudo -i
apt-get update
ap-get install stress -y
stress -c 5

ASG should automatically add a new instance when CPU utilization exceeds threshold
Refresh ALB DNS → new instance hostname should appear


## 📸 Screenshots
| Description             | Image |
|-------------------------|-------|
| Auto Scaling Group       | ![ASG](screenshots/asg-created.png) |
| Load Balancer Listener   | ![ALB](screenshots/alb-listener.png) |
| Dynamic Scaling Test     | ![Scaling](screenshots/scaling-test.png) |
| EC2 Webpage              | ![Webpage](screenshots/nginx-instance.png) |


##Step 6: Cleanup

Delete ASG, Launch Template, ALB, Target Group, and Security Groups


