# 🌐 QuickLoan — PHP Web Application on AWS

A scalable, cloud-hosted loan application platform built with **PHP + Apache/Nginx** on **AWS**, featuring a full custom VPC setup, EC2-based deployment, RDS MySQL database, and S3-based media storage — fully automated with **AWS CloudFormation**.

***

## 📐 Architecture Overview

```
Internet
   │
   ▼
Internet Gateway (IGW)
   │
   ▼
VPC (172.20.0.0/16)
   ├── Public Subnet 1 (AZ1) ── EC2 (PHP + Nginx)
   ├── Public Subnet 2 (AZ2)
   └── Public Subnet 3 (AZ3)
         │
         ├── Security Group (Port 22, 80)
         ├── Route Table → IGW
         ├── RDS MySQL (quickloandb)
         └── S3 Bucket (images & logos)
```

***

## ☁️ AWS Services Used

| Service | Purpose |
|---|---|
| **VPC** | Custom isolated network (172.20.0.0/16) |
| **Subnets (x3)** | Public subnets across 3 Availability Zones |
| **Internet Gateway** | Allows internet access to the VPC |
| **Route Table** | Routes 0.0.0.0/0 traffic to IGW |
| **Security Group** | Allows inbound SSH (22), HTTP (80); all egress |
| **EC2 Instance** | Hosts the PHP + Nginx web application |
| **RDS (MySQL)** | Managed relational database (`quickloandb`) |
| **S3 Bucket** | Stores application images and logos |
| **CloudFormation** | Infrastructure as Code (IaC) for full stack provisioning |

***

## 🗂️ Project Structure

```
quickloan/
├── vpc-ec2.yml            # CloudFormation template (VPC, Subnets, IGW, RT, SG, EC2)
├── init.sql               # SQL script to create database and table
├── commands.txt           # Step-by-step EC2 setup commands
├── public/                # Frontend (HTML, CSS, JS)
├── includes/
│   └── dbconnect.php      # RDS connection config
└── quickloan.conf         # Nginx virtual host config
```

***

## 🗄️ Database Schema

```sql
CREATE DATABASE quickloandb;
USE quickloandb;

CREATE TABLE applications (
    id         INT AUTO_INCREMENT PRIMARY KEY,
    name       VARCHAR(255) NOT NULL,
    contact    VARCHAR(20)  NOT NULL,
    email      VARCHAR(255) NOT NULL,
    loantype   VARCHAR(50)  NOT NULL
);
```

***

## 🚀 Deployment Guide

### Step 1 — Launch Infrastructure via CloudFormation

```bash
aws cloudformation deploy \
  --template-file vpc-ec2.yml \
  --stack-name quickloan-stack \
  --capabilities CAPABILITY_IAM
```

> 💡 Set `UseSsmAmi=true` to automatically use the latest Amazon Linux 2023 AMI.

***

### Step 2 — Connect to EC2 & Install Dependencies

```bash
# Set hostname
sudo hostnamectl set-hostname appsrv

# Update packages
sudo dnf update -y

# Install Nginx
sudo dnf install nginx -y
sudo systemctl start nginx && sudo systemctl enable nginx

# Install PHP 8.2
sudo dnf install php8.2 php-fpm php-mysqlnd php-pdo php-mbstring -y
sudo systemctl start php-fpm && sudo systemctl enable php-fpm

# Install MariaDB client (for RDS access)
sudo yum install mariadb105 -y
```

***

### Step 3 — Deploy Application Files

```bash
# Copy app files to Nginx web root
sudo cp -r includes public /usr/share/nginx/html/

# Set correct permissions
sudo chown -R nginx:nginx /usr/share/nginx/html/public
sudo chmod -R 755 /usr/share/nginx/html/public

# Deploy Nginx config
sudo cp quickloan.conf /etc/nginx/conf.d/
sudo nginx -t
sudo systemctl restart nginx
```

***

### Step 4 — Configure RDS Connection

Edit `includes/dbconnect.php` and fill in your RDS details:

```php
$db_endpoint = "your-rds-endpoint";
$username    = "your-username";
$password    = "your-password";
$dbname      = "quickloandb";
```

***

### Step 5 — Initialize the Database

```bash
mysql -h <RDS_ENDPOINT> -u <USERNAME> -p < init.sql
```

***

### Step 6 — Configure S3 for Media Storage

- Create an S3 bucket (e.g., `quickloan-assets`)
- Upload images and logos to the bucket
- Update your application code to reference the S3 bucket URL for media files

***

## 🔒 Security Group Rules

| Type | Protocol | Port | Source |
|---|---|---|---|
| SSH | TCP | 22 | 0.0.0.0/0 *(restrict in prod)* |
| HTTP | TCP | 80 | 0.0.0.0/0 |
| All Egress | All | All | 0.0.0.0/0 |

> ⚠️ For production, restrict SSH access to your IP range only.

***

## ⚙️ CloudFormation Parameters

| Parameter | Default | Description |
|---|---|---|
| `VPCCIDRBlock` | `172.20.0.0/16` | VPC CIDR block |
| `Subnet1CIDRBlock` | `172.20.1.0/24` | Subnet 1 (AZ1) |
| `Subnet2CIDRBlock` | `172.20.2.0/24` | Subnet 2 (AZ2) |
| `Subnet3CIDRBlock` | `172.20.3.0/24` | Subnet 3 (AZ3) |
| `InstanceType` | `t2.small` | EC2 instance type |
| `UseSsmAmi` | `false` | Use latest AL2023 AMI via SSM |
| `CustomAmiId` | `ami-0f559c3138` | Custom AMI ID (if UseSsmAmi=false) |

***

## 📦 Tech Stack

- **Backend:** PHP 8.2 + Apache/Nginx
- **Frontend:** HTML, CSS, JavaScript
- **Database:** Amazon RDS (MySQL)
- **Storage:** Amazon S3
- **Server:** Amazon EC2 (Amazon Linux 2023)
- **Networking:** AWS VPC, Subnets, IGW, Route Tables, Security Groups
- **IaC:** AWS CloudFormation

***

## 📸 Application Flow

```
User Request
    │
    ▼
EC2 (Nginx + PHP-FPM)
    │         │
    ▼         ▼
RDS MySQL   S3 Bucket
(Form Data) (Images/Logos)
```

***

## 🙋 Author

**[Your Name]**  
AWS Cloud & DevOps Enthusiast  
[LinkedIn](https://www.linkedin.com) · [GitHub](https://www.github.com)

***



