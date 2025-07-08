# 🌐 Web Server Setup Guide for AWS Three-Tier Architecture

This document provides detailed instructions for setting up the **Web Tier** of your AWS Three-Tier Architecture. It includes steps for copying the web app code, installing Node.js for React, building the app, and configuring NGINX as a web server and reverse proxy.

---

## ☁️ 1. Copy Web-Tier Code from S3

Ensure your instance has the correct IAM role with `AmazonS3ReadOnlyAccess` permission.

```bash
#!/bin/bash

# Switch to ec2-user and go to home directory
sudo -su ec2-user
cd /home/ec2-user

# Replace <YOUR-S3-BUCKET-NAME> with your actual S3 bucket
sudo aws s3 cp s3://<YOUR-S3-BUCKET-NAME>/application-code/web-tier web-tier --recursive

# Set proper ownership and permissions
sudo chown -R ec2-user:ec2-user /home/ec2-user
sudo chmod -R 755 /home/ec2-user
```

---

## 🛠️ 2. Install Node.js (Required for React App)

**Reference**: [AWS Node.js Setup](https://docs.aws.amazon.com/sdk-for-javascript/v2/developer-guide/setting-up-node-on-ec2-instance.html)

```bash
# Install Node Version Manager (NVM)
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
source ~/.bashrc

# Install and use Node.js version 16
nvm install 16
nvm use 16

# Go to the web-tier project folder and install dependencies
cd /home/ec2-user/web-tier
npm install

# Optional: Fix vulnerabilities
# npm audit fix
```

---

## 🏗️ 3. Build the App for Production

React apps need to be compiled into static files to be served by NGINX.

```bash
cd /home/ec2-user/web-tier
npm run build
```

This command generates a `build/` folder containing static HTML, CSS, and JS files.

---

## 🌐 4. Install and Configure NGINX

NGINX will serve the static React files and act as a reverse proxy to the internal load balancer for API calls.

**Reference**: [Install and Configure NGINX on Amazon Linux 2023](https://dev.to/0xfedev/how-to-install-nginx-as-reverse-proxy-and-configure-certbot-on-amazon-linux-2023-2cc9)

```bash
# Install NGINX
sudo yum install nginx -y

# Backup the default configuration
cd /etc/nginx
sudo mv nginx.conf nginx-backup.conf

# Replace with the updated nginx.conf (with Internal-ALB DNS added)
# Replace <YOUR-S3-BUCKET-NAME> with your actual S3 bucket name
sudo aws s3 cp s3://<YOUR-S3-BUCKET-NAME>/application-code/nginx.conf .

# Set proper permissions and restart the web server
sudo chmod -R 755 /home/ec2-user
sudo service nginx restart

# Enable NGINX to start on boot
sudo chkconfig nginx on
```

---

## ✅ Summary

You have now:

- Downloaded and prepared your React web application from S3
- Installed Node.js and built the app for production
- Configured NGINX to serve the app and forward API calls to the internal load balancer

Your Web Tier is now ready and serving traffic in your AWS Three-Tier Architecture!
