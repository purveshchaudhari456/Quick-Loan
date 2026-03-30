# 🏦 QuickLoan — Scalable PHP Web Application on AWS

> A production-grade, cloud-hosted loan application platform built on AWS with auto-scaling, load balancing, RDS, and S3 — deployed in the **Asia Pacific (Mumbai) `ap-south-1`** region.

---

## 🌐 Live Application

| Page | Description |
|------|-------------|
| Home | Loan products — Home, Gold, Vehicle, Personal Loans |
| Apply for Loan | Form collecting Name, Contact, Email, Loan Type |

> Application served via **ALB DNS**: `LB1-41819127.ap-south-1.elb.amazonaws.com`

---

## 📸 Project Screenshots

### Application UI
| Home Page | Loan Products | Apply for Loan Form |
|-----------|--------------|---------------------|
| ![Home](screenshots/home.png) | ![Products](screenshots/products.png) | ![Form](screenshots/form.png) |

### AWS Infrastructure
| VPC & Subnets | EC2 Instances | Auto Scaling Group |
|---------------|--------------|-------------------|
| ![VPC](screenshots/vpc.png) | ![EC2](screenshots/ec2.png) | ![ASG](screenshots/asg.png) |

| Load Balancer | RDS Database | S3 Bucket |
|---------------|-------------|-----------|
| ![ALB](screenshots/alb.png) | ![RDS](screenshots/rds.png) | ![S3](screenshots/s3.png) |

---

## 📐 Architecture Overview

```
                        ┌─────────────────────────────────────────┐
                        │         AWS Cloud — ap-south-1          │
                        │                                         │
   Users                │   ┌─────────────────────────────────┐   │
     │                  │   │       VPC: 172.20.0.0/16        │   │
     ▼                  │   │                                 │   │
 Internet               │   │  ┌──────────────────────────┐  │   │
     │                  │   │  │   Internet Gateway (IGW) │  │   │
     ▼                  │   │  └────────────┬─────────────┘  │   │
 ┌───────┐              │   │               │ Route Table     │   │
 │  ALB  │◄─────────────┼───┼───── 0.0.0.0/0 → IGW ─────────┤   │
 │  LB1  │  Internet-   │   │                                 │   │
 │       │  facing      │   │  Subnet-1    Subnet-2  Subnet-3 │   │
 └───┬───┘              │   │  AZ(1a)      AZ(1b)    AZ(1c)   │   │
     │                  │   │  .1.0/24     .2.0/24   .3.0/24  │   │
     ▼                  │   │      │           │               │   │
 ┌──────────────────┐   │   │  ┌───┴───┐   ┌──┴────┐          │   │
 │  Auto Scaling    │   │   │  │  EC2  │   │  EC2  │ ← ASG    │   │
 │  Group: asg1     │   │   │  │t2.sml │   │t2.sml │          │   │
 │  Min:2  Max:4    │   │   │  └───────┘   └───────┘          │   │
 └──────────────────┘   │   │           │                      │   │
                        │   │           ▼                      │   │
 ┌──────────────────┐   │   │    ┌─────────────┐              │   │
 │   S3: qloan      │   │   │    │  RDS MySQL  │              │   │
 │  loan images     │   │   │    │    db1      │              │   │
 │  + logo          │   │   │    │ db.t3.micro │              │   │
 └──────────────────┘   │   │    └─────────────┘              │   │
                        │   └─────────────────────────────────┘   │
                        └─────────────────────────────────────────┘
```

---

## ☁️ AWS Infrastructure — Full Stack

| # | Component | Name | Details |
|---|-----------|------|---------|
| 1 | **VPC** | `MyVPC` | CIDR: `172.20.0.0/16`, ap-south-1 |
| 2 | **Subnets** | Subnet1-AZ1, 2, 3 | 3 public subnets across `1a`, `1b`, `1c` |
| 3 | **Route Table** | `CustomPublicRouteTable` | 3 subnet associations, 2 routes |
| 4 | **Internet Gateway** | `MyInternetGateway` | Internet routes to 3 public subnets |
| 5 | **Route** | `0.0.0.0/0` | Points to IGW |
| 6 | **Security Group** | `InstanceSecurityGroup` | Inbound: 22, 80 — Outbound: All |
| 7 | **EC2 Instances** | `VM1` + 2 ASG instances | `t2.small`, Amazon Linux 2023 |
| 8 | **RDS** | `db1` | MySQL 8.4.7, `db.t3.micro`, ap-south-1c |
| 9 | **Auto Scaling Group** | `asg1` | Min: 2, Desired: 2, Max: 4, 3 AZs |
| 10 | **Launch Template** | `Quickloan-web-1` | Version Default |
| 11 | **Load Balancer** | `LB1` | ALB, Internet-facing, IPv4, 3 AZs |
| 12 | **S3 Bucket** | `qloan` | Stores loan product images + logo |

---

## 🗂️ Project Structure

```
quickloan/
├── vpc-ec2.yml              # CloudFormation template (VPC + EC2)
├── init.sql                 # Database schema
├── commands.txt             # Server setup commands
├── public/                  # Frontend (HTML, CSS, JS)
├── includes/
│   └── db_connect.php       # RDS connection config
└── nginx/
    └── quickloan.conf       # Nginx virtual host config
```

---

## 🗄️ Database Schema

```sql
CREATE DATABASE quickloan_db;
USE quickloan_db;

CREATE TABLE applications (
    id         INT AUTO_INCREMENT PRIMARY KEY,
    name       VARCHAR(255) NOT NULL,
    contact    VARCHAR(20)  NOT NULL,
    email      VARCHAR(255) NOT NULL,
    loan_type  VARCHAR(50)  NOT NULL
);
```

**Sample data stored in RDS:**

| id | name | contact | email | loan_type |
|----|------|---------|-------|-----------|
| 1 | ALAM | 97873876853 | ALAM@GMAIL.COM | Gold Loan |
| 2 | ajay | 683687538758 | ajay@gmail.com | Personal Loan |
| 3 | ajay | 7838683686 | ajay@gmail.com | Vehicle Loan |

---

## 🚀 Deployment Guide

### Step 1 — Deploy Infrastructure via CloudFormation

```bash
aws cloudformation deploy \
  --template-file vpc-ec2.yml \
  --stack-name quickloan-stack \
  --parameter-overrides InstanceType=t2.small
```

### Step 2 — Configure the EC2 Instance

```bash
sudo hostnamectl set-hostname appsrv
sudo dnf update -y

# Nginx
sudo dnf install nginx -y
sudo systemctl start nginx && sudo systemctl enable nginx

# PHP 8.2
sudo dnf install php8.2 php-fpm php-mysqlnd php-pdo php-mbstring -y
sudo systemctl start php-fpm && sudo systemctl enable php-fpm
sudo systemctl restart nginx

# MySQL client
sudo yum install mariadb105 -y
```

### Step 3 — Connect to RDS & Initialize Database

```bash
mysql -h <rds-endpoint> -P 3306 -u admin -p
```

Then run `init.sql` to create the database and table.

### Step 4 — Deploy App Files

```bash
sudo cp -r public/   /usr/share/nginx/html/
sudo cp -r includes/ /usr/share/nginx/html/
sudo chown -R nginx:nginx /usr/share/nginx/html/public
sudo chmod -R 755 /usr/share/nginx/html/public
sudo cp /usr/share/nginx/html/nginx/quickloan.conf /etc/nginx/conf.d/
sudo nginx -t && sudo systemctl restart nginx
```

### Step 5 — Configure DB Connection

Edit `includes/db_connect.php`:

```php
$host     = "YOUR_RDS_ENDPOINT";
$username = "admin";
$password = "YOUR_PASSWORD";
$dbname   = "quickloan_db";
```

### Step 6 — Create AMI → Launch Template → Auto Scaling Group

1. Configure your EC2 instance fully → **Create AMI** from it
2. Create **Launch Template** `Quickloan-web-1` using the AMI
3. Create **Auto Scaling Group** `asg1`:
   - Min: `2` | Desired: `2` | Max: `4`
   - Subnets: All 3 AZs (`ap-south-1a`, `1b`, `1c`)
   - Attach to Load Balancer `LB1`

### Step 7 — Configure Application Load Balancer

- Type: **Application Load Balancer (ALB)**
- Scheme: **Internet-facing**
- Subnets: All 3 AZs
- Target Group: EC2 instances managed by ASG

---

## 🪣 S3 Static Assets

Bucket `qloan` stores loan product images served to the frontend:

| File | Size |
|------|------|
| `gold_loan.jpg` | 142.6 KB |
| `home_loan.jpg` | 26.7 KB |
| `personal_loan.jpg` | 117.0 KB |
| `vehicle_loan.jpg` | 224.1 KB |
| `quickloan_logo.png` | 16.7 KB |

---

## 🛠️ Tech Stack

| Layer | Technology |
|-------|------------|
| Cloud | AWS (VPC, EC2, RDS, ALB, ASG, S3) |
| IaC | AWS CloudFormation |
| OS | Amazon Linux 2023 |
| Web Server | Nginx |
| Backend | PHP 8.2 |
| Database | MySQL 8.4.7 (RDS `db.t3.micro`) |
| Frontend | HTML, CSS, JavaScript |
| Region | ap-south-1 (Mumbai) |

---

## 📈 Scalability & High Availability

- **Auto Scaling Group** automatically adds/removes EC2 instances based on demand (Min 2 → Max 4)
- **ALB** distributes traffic across instances in **3 Availability Zones**
- **RDS** managed database with automated backups
- **S3** for static asset storage — decoupled from EC2

---

## 🔒 Security Group Rules

| Type | Port | Source | Purpose |
|------|------|--------|---------|
| Inbound TCP | 22 | 0.0.0.0/0 | SSH *(restrict to your IP in production)* |
| Inbound TCP | 80 | 0.0.0.0/0 | HTTP web traffic |
| Outbound All | All | 0.0.0.0/0 | All outbound |

---

> *Built as a hands-on cloud infrastructure project demonstrating VPC design, EC2 provisioning, RDS integration, Auto Scaling, Load Balancing, and S3 static asset storage on AWS — Region: ap-south-1 (Mumbai).*
