# 🛠️ Application Server Setup Guide for AWS Three-Tier Architecture

This guide provides step-by-step commands for configuring and deploying the **Application Tier** of your AWS three-tier architecture. It focuses on installing MySQL client, copying application code from S3, setting up Node.js, and managing the Node.js app using PM2 on **Amazon Linux 2023**.

---

## 📦 1. Install MySQL Client on Amazon Linux 2023

This setup enables the application server to connect to the Aurora MySQL database.

**Reference**: [Install MySQL on Amazon Linux 2023](https://dev.to/aws-builders/installing-mysql-on-amazon-linux-2023-1512)

```bash
#!/bin/bash

# Switch to ec2-user (if not already)
sudo -su ec2-user

# Download and install MySQL 8.0 community release
sudo wget https://dev.mysql.com/get/mysql80-community-release-el9-1.noarch.rpm
sudo dnf install mysql80-community-release-el9-1.noarch.rpm -y

# Import GPG key for secure installation
sudo rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2023

# Install MySQL client
sudo dnf install mysql-community-client -y

# Verify installation
mysql --version
```

### ✅ Test Database Connectivity

Replace `<RDS-Endpoint>` and `<username>` with your actual RDS values:

```bash
mysql -h <RDS-Endpoint> -u <username> -p
# Press Enter and provide your password
```

---

## ☁️ 2. Copy Application Code from S3 Bucket

Ensure your IAM role has `AmazonS3ReadOnlyAccess` permissions.

```bash
# Navigate to home directory
cd /home/ec2-user

# Replace with your actual S3 bucket name
sudo aws s3 cp s3://<YOUR-S3-BUCKET-NAME>/application-code/app-tier app-tier --recursive

# Set proper ownership and permissions
cd app-tier
sudo chown -R ec2-user:ec2-user /home/ec2-user/app-tier
sudo chmod -R 755 /home/ec2-user/app-tier
```

---

## 🌐 3. Install Node.js Using NVM (Node Version Manager)

**Reference**: [AWS SDK for JavaScript Setup](https://docs.aws.amazon.com/sdk-for-javascript/v2/developer-guide/setting-up-node-on-ec2-instance.html)

```bash
# Install NVM
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
source ~/.bashrc

# Install and use Node.js v16
nvm install 16
nvm use 16

# Install PM2 process manager globally
npm install -g pm2

# Install app dependencies and fix vulnerabilities
npm install
npm audit fix
```

---

## 🚀 4. Run and Manage the Application

Start the Node.js app (`index.js`) using PM2 and configure it to run at boot.

```bash
# Start the app with PM2
pm2 start index.js

# View logs (use Ctrl+C to exit)
pm2 logs

# Set PM2 to launch at system startup
pm2 startup

# Generate and configure startup command
sudo env PATH=$PATH:/home/ec2-user/.nvm/versions/node/v16.20.2/bin \
  /home/ec2-user/.nvm/versions/node/v16.20.2/lib/node_modules/pm2/bin/pm2 \
  startup systemd -u ec2-user --hp /home/ec2-user

# Save current PM2 process list
pm2 save
```

---

## ❤️ 5. Application Health Check

Run this to confirm the app is up and running:

```bash
curl http://localhost:4000/health
```

You should see a response confirming the service is healthy.

---

## ✅ Summary

You have now:

- Installed and tested MySQL client
- Fetched and configured app-tier code from S3
- Installed Node.js and dependencies
- Deployed and managed your app with PM2

Your application server is ready and fully integrated into the AWS three-tier environment.
