# 🏗️ AWS Three-Tier Web-Site Deployment Project

## 📘 Overview

This hands-on workshop walks you through building a fully functional **three-tier web architecture on AWS**. You'll manually create and configure all necessary components—networking, computing, security, and database—to ensure high **availability**, **scalability**, and **resilience** of the application infrastructure.

By the end of this workshop, you will understand how to build a secure, production-grade cloud environment using core AWS services and deploy a sample React/Node.js application with an Aurora MySQL backend.

---

## 🧭 Architecture Overview

! [AWS Architecture Diagram](https://drive.google.com/file/d/1Vj4owcFLKJj7GeEBa-rOJYjifMZBvjgs/view?usp=sharing)
<img title="3-Tier-Architecture" alt="Alt text" src="https://drive.google.com/file/d/1Vj4owcFLKJj7GeEBa-rOJYjifMZBvjgs/view?usp=sharing">

### Key Components:

- **Web Tier (Frontend)**: Public-facing EC2 instances running Nginx and serving a React.js application.
- **Application Tier (Backend)**: Private EC2 instances running a Node.js API behind an internal Load Balancer.
- **Database Tier**: A highly available Aurora MySQL cluster deployed across multiple AZs.
- **Load Balancing**: Both external (public) and internal (private) Application Load Balancers (ALBs).
- **Autoscaling Groups (ASG)**: Maintain elasticity and health of web and application layers.
- **Security Groups**: Layered security from front to database.
- **Monitoring and Logging**: VPC flow logs, CloudWatch alarms, and CloudTrail for auditing.

---

## 🔁 Step-by-Step Implementation Guide

### Step 1: 📥 Download Code Repository

Clone or download the required application code (web + app servers) from GitHub onto your local machine.

```bash
git clone  https://github.com/vitthalSanadhya/3-Tire-AWS-Project.git
```

---

### Step 2: 🪣 Create S3 Buckets

- **S3 Bucket #1**: For storing and retrieving web and application server code.
- **S3 Bucket #2**: For capturing **VPC Flow Logs**.

Upload the downloaded code to S3 Bucket #1.

---

### Step 3: 🛡️ IAM Role Creation

Create an IAM Role with the following **policies**:

- **AmazonS3ReadOnlyAccess** – To allow EC2 instances to read application code from S3.
- **AmazonSSMManagedInstanceCore** – For EC2 Systems Manager access.

Attach this role to both web and app EC2 instances.

---

### Step 4: 🌐 Create Networking Components

- **VPC**: With appropriate CIDR blocks.
- **Subnets**: At least 2 public and 2 private subnets in different AZs.
- **Internet Gateway (IGW)**: Attach to the VPC for public subnet access.
- **NAT Gateway**: For outbound internet access from private subnets.
- **Route Tables (RT)**: Configure appropriate routes for public and private subnets.

📝 **Tip**: Enable _Auto-Assign Public IP_ for EC2s in public subnets.

Also, enable **VPC Flow Logs** and configure them to deliver logs to your VPC logs S3 bucket.

---

### Step 5: 🔐 Set Up Security Groups

| SG Name                   | Protocol / Port     | Source         |
| ------------------------- | ------------------- | -------------- |
| External-Load-Balancer-SG | HTTP (80)           | `0.0.0.0/0`    |
| Web-Tier-SG               | HTTP (80)           | External-LB-SG |
| Internal-Load-Balancer-SG | HTTP (80)           | Web-Tier-SG    |
| App-Tier-SG               | Custom TCP (4000)   | Internal-LB-SG |
| DB-Tier-SG                | MySQL/Aurora (3306) | App-Tier-SG    |

---

### Step 6: 🗄️ Database Setup (Aurora MySQL)

- **Create a DB Subnet Group**: Include private subnets.
- **Launch Aurora MySQL Cluster**:
  - Choose Multi-AZ deployment.
  - Attach DB Subnet Group.
  - Associate DB-Tier-SG security group.

---

### Step 7: 🧪 Build & Deploy Application Tier

1. Launch a test EC2 instance in a private subnet.
2. Install Node.js and other dependencies.
3. Run application tests and validate DB connectivity.
4. [Use these setup commands](https://github.com/vitthalSanadhya/3-Tire-AWS-Project/main/app-server-commands).
5. Create an **AMI** from this instance.
6. Create:
   - Launch Template (using the AMI)
   - Target Group (App Tier)
   - Internal ALB
   - Auto Scaling Group for application layer
7. Update `nginx.conf` (to include internal ALB DNS) and upload it to S3.

---

### Step 8: 🧪 Build & Deploy Web Tier

1. Launch a test EC2 instance in a public subnet.
2. Install Nginx, Node.js, and React.js.
3. Test the frontend and ensure it proxies API requests to the app tier.
4. [Use these setup commands](https://github.com/vitthalSanadhya/3-Tire-AWS-Project/main/web-server-commands).
5. Create an **AMI** from this instance.
6. Create:
   - Launch Template (using the AMI)
   - Target Group (Web Tier)
   - External ALB
   - Auto Scaling Group for web layer

---

### Step 9: 🌐 Route 53 DNS Configuration

- Create a DNS record pointing your domain to the **External ALB DNS** name.

---

### Step 10: 🔔 Monitoring with CloudWatch

- Set up **CloudWatch Alarms** for metrics like:
  - High CPU
  - Unhealthy instance count
- Create **SNS topics** to notify admins when alarms are triggered.

---

### Step 11: 📜 Enable AWS CloudTrail

- Enable **CloudTrail** in your region for audit logs of all API activities.

---

### Step 12: 🧹 Clean-Up Resources

To avoid unnecessary billing, follow this teardown order:

1. Delete **CloudFront** distributions (if used).
2. Delete **CloudWatch Alarms** and **SNS Topics**.
3. Remove **Route 53 records**.
4. Delete:
   - ALBs (External/Internal)
   - Target Groups
   - ASGs
   - Launch Templates
5. Delete **Security Groups**
6. Delete **NAT Gateway** (wait ~5 minutes)
7. Release **Elastic IPs**
8. Delete **VPC**
9. Remove **RDS & DB Subnet Group**
10. Delete **S3 Buckets**

---

## 📚 Reference & Workshop Portal

Access full workshop content and interactive instructions here:

👉 [AWS Three-Tier Web Architecture Workshop (AWS Events Catalog)](https://catalog.us-east-1.prod.workshops.aws/workshops/85cd2bb2-7f79-4e96-bdee-8078e469752a/en-US)
