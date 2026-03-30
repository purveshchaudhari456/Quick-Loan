<div align="center">

<h1>⚡ QuickLoan — Scalable Web Application on AWS</h1>

<p>A PHP-based loan application platform deployed on a fully production-grade AWS infrastructure with VPC, EC2, RDS, S3, Auto Scaling Group, and Application Load Balancer.</p>

<p>
  <img src="https://img.shields.io/badge/PHP-8.2-777BB4?style=for-the-badge&logo=php&logoColor=white"/>
  <img src="https://img.shields.io/badge/MySQL-8.0-4479A1?style=for-the-badge&logo=mysql&logoColor=white"/>
  <img src="https://img.shields.io/badge/Nginx-1.x-009639?style=for-the-badge&logo=nginx&logoColor=white"/>
  <img src="https://img.shields.io/badge/AWS-EC2-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white"/>
  <img src="https://img.shields.io/badge/AWS-RDS-527FFF?style=for-the-badge&logo=amazonaws&logoColor=white"/>
  <img src="https://img.shields.io/badge/AWS-S3-569A31?style=for-the-badge&logo=amazons3&logoColor=white"/>
  <img src="https://img.shields.io/badge/Auto_Scaling-Active-orange?style=for-the-badge&logo=amazonaws&logoColor=white"/>
  <img src="https://img.shields.io/badge/Load_Balancer-ALB-FF4F8B?style=for-the-badge&logo=amazonaws&logoColor=white"/>
  <img src="https://img.shields.io/badge/Region-ap--south--1-232F3E?style=for-the-badge&logo=amazonaws&logoColor=white"/>
  <img src="https://img.shields.io/badge/License-MIT-green?style=for-the-badge"/>
</p>

<p>
  <a href="#-live-application">Live App</a> •
  <a href="#-screenshots">Screenshots</a> •
  <a href="#-architecture">Architecture</a> •
  <a href="#-aws-infrastructure">AWS Infrastructure</a> •
  <a href="#-deployment">Deployment</a>
</p>

</div>

---

## 🌐 Live Application

> The application is deployed and running live on AWS EC2 behind an Application Load Balancer.
>
> **ALB DNS:** `LB1-41819127.ap-south-1.elb.amazonaws.com`

---

## 📸 Screenshots

### Home Page — Loan Products
![QuickLoan Home Page](docs/screenshots/home.png)

### Loan Products Section
![Loan Products](docs/screenshots/products.png)

### Loan Application Form
![Apply for Loan](docs/screenshots/apply.png)

### AWS Infrastructure — EC2 Instances (Auto Scaled)
![EC2 Instances](docs/screenshots/ec2-instances.png)

### Auto Scaling Group
![Auto Scaling Group](docs/screenshots/asg.png)

### Application Load Balancer
![Load Balancer](docs/screenshots/alb.png)

### Amazon RDS — MySQL Database
![RDS Database](docs/screenshots/rds.png)

### VPC — Subnets & Route Tables
![VPC](docs/screenshots/vpc.png)

### S3 Bucket — Static Assets
![S3 Bucket](docs/screenshots/s3.png)

### Live DB — Applications Table (via EC2 SSH)
![Database Records](docs/screenshots/db-records.png)

---

## 🚀 Features

- 🏠 **Loan Products Page** — Home Loans, Gold Loans, Vehicle Loans, Personal Loans with images served from S3
- 📋 **Online Loan Application Form** — Collects name, contact, email, and loan type; saved to RDS
- 🗄️ **Amazon RDS MySQL** — Managed database (`db.t3.micro`) in `ap-south-1c` private subnet
- 🖼️ **Amazon S3** — Bucket `qloan` stores all product images and the logo
- 🌐 **Nginx + PHP 8.2** — Production web server on Amazon Linux 2023
- ☁️ **Infrastructure as Code** — Full AWS stack via CloudFormation (`vpc-ec2.yml`)
- 📈 **Auto Scaling Group** (`asg1`) — Min: 2, Max: 4 instances across 3 Availability Zones
- ⚖️ **Application Load Balancer** (`LB1`) — Internet-facing ALB, Active, distributes across AZ-1a, AZ-1b, AZ-1c
- 🔒 **Secure VPC Architecture** — Custom VPC with separate Security Groups for EC2 and RDS

---

## 🏗️ Architecture

```
                    ┌────────────────────────────────────────────────────────────┐
                    │                  AWS  ap-south-1  (Mumbai)                 │
                    │                                                            │
                    │   ┌──────────────────────────────────────────────────┐    │
                    │   │              MyVPC  (172.20.0.0/16)              │    │
                    │   │                                                  │    │
                    │   │   PUBLIC SUBNETS                                 │    │
                    │   │   ┌──────────────┐ ┌──────────────┐ ┌────────┐  │    │
                    │   │   │ Subnet1-AZ1  │ │ Subnet2-AZ2  │ │Sub3-AZ3│  │    │
                    │   │   │172.20.1.0/24 │ │172.20.2.0/24 │ │.3.0/24 │  │    │
                    │   │   └──────┬───────┘ └──────┬────────┘ └───┬────┘  │    │
                    │   │          │                 │              │       │    │
                    │   │   ┌──────▼─────────────────▼──────────────▼────┐  │    │
                    │   │   │     Application Load Balancer  (LB1)      │  │    │
                    │   │   │      Internet-facing  |  Port 80/443      │  │    │
                    │   │   └──────────────────────┬─────────────────────┘  │    │
                    │   │                          │                        │    │
                    │   │   ┌───────────────────────▼─────────────────────┐ │    │
                    │   │   │         Auto Scaling Group  (asg1)          │ │    │
                    │   │   │   Launch Template: Quickloan-web-1          │ │    │
                    │   │   │   Min: 2  |  Desired: 2  |  Max: 4          │ │    │
                    │   │   │  ┌──────────────┐   ┌──────────────┐        │ │    │
                    │   │   │  │  EC2  AZ-1a  │   │  EC2  AZ-1b  │  ...  │ │    │
                    │   │   │  │  Nginx       │   │  Nginx       │       │ │    │
                    │   │   │  │  PHP 8.2     │   │  PHP 8.2     │       │ │    │
                    │   │   │  └──────┬───────┘   └──────┬───────┘       │ │    │
                    │   │   └─────────┼─────────────────┼────────────────┘ │    │
                    │   │             │                 │                   │    │
                    │   │   PRIVATE SUBNET              │                   │    │
                    │   │   ┌──────────▼─────────────────▼──────────────┐  │    │
                    │   │   │       Amazon RDS  db1  (MySQL 8.0)        │  │    │
                    │   │   │       db.t3.micro  |  ap-south-1c         │  │    │
                    │   │   │       quickloan_db  |  Port 3306          │  │    │
                    │   │   └───────────────────────────────────────────┘  │    │
                    │   │                                                  │    │
                    │   │   ┌────────────────────────────────────────────┐ │    │
                    │   │   │         Internet Gateway (MyIGW)           │ │    │
                    │   │   └───────────────────┬────────────────────────┘ │    │
                    │   └───────────────────────┼──────────────────────────┘    │
                    │                           │                               │
                    │   ┌───────────────────────▼───────────────────────────┐   │
                    │   │        Amazon S3  Bucket: qloan                   │   │
                    │   │  gold_loan.jpg | home_loan.jpg | vehicle_loan.jpg │   │
                    │   │  personal_loan.jpg | quickloan_logo.png           │   │
                    │   └───────────────────────────────────────────────────┘   │
                    └────────────────────────────────────────────────────────────┘
                                                │
                                           [ Internet ]
                                                │
                                              Users
```

---

## ☁️ AWS Infrastructure

| # | Component | Name / ID | Details |
|---|---|---|---|
| 1 | **VPC** | `MyVPC` | `172.20.0.0/16` — ap-south-1 |
| 2 | **Subnets** | Subnet1-AZ1, Subnet2-AZ2, Subnet3-AZ3 | 3 public subnets across all AZs |
| 3 | **Route Table** | `CustomPublicRouteTable` | 3 subnet associations, 2 routes |
| 4 | **Internet Gateway** | `MyInternetGateway` | Routes internet to 3 public subnets |
| 5 | **Route** | `0.0.0.0/0 → IGW` | Outbound internet for all public subnets |
| 6 | **Security Groups** | `InstanceSecurityGroup` | EC2: ports 22 & 80 · RDS: port 3306 from EC2 only |
| 7 | **EC2 Instances** | `VM1` + ASG instances | Amazon Linux 2023, `t2.small`, Nginx + PHP 8.2 |
| 8 | **Amazon RDS** | `db1` | MySQL 8.0, `db.t3.micro`, ap-south-1c, Status: Available |
| 9 | **S3 Bucket** | `qloan` | Stores product images & logo (5 objects) |
| 10 | **Launch Template** | `Quickloan-web-1` | AMI of tested EC2 instance |
| 11 | **Auto Scaling Group** | `asg1` | Min: 2, Desired: 2, Max: 4, across 3 AZs |
| 12 | **Load Balancer** | `LB1` | Application LB, Internet-facing, IPv4, Active |

### Security Group Rules

**EC2 Security Group**

| Type | Protocol | Port | Source |
|---|---|---|---|
| SSH inbound | TCP | 22 | 0.0.0.0/0 *(restrict to your IP in prod)* |
| HTTP inbound | TCP | 80 | 0.0.0.0/0 |
| All outbound | All | All | 0.0.0.0/0 |

**RDS Security Group**

| Type | Protocol | Port | Source |
|---|---|---|---|
| MySQL inbound | TCP | 3306 | EC2 Security Group only |
| All outbound | All | All | 0.0.0.0/0 |

---

## 🗂️ Project Structure

```
Quick-Loan/
├── public/                    # Web root served by Nginx
│   ├── index.php              # Home page — loan products, hero section
│   └── apply.php              # Loan application form
├── includes/                  # Shared PHP components
│   ├── db_connect.php         # RDS MySQL connection config
│   ├── header.php             # Common page header + logo
│   └── footer.php             # Common page footer
├── nginx/
│   └── quickloan.conf         # Nginx virtual host configuration
├── init.sql                   # Database schema (run once on RDS)
├── vpc-ec2.yml                # AWS CloudFormation IaC template
└── README.md
```

---

## 🛠️ Tech Stack

| Layer | Technology |
|---|---|
| Frontend | HTML5, CSS3, JavaScript |
| Backend | PHP 8.2 + PHP-FPM |
| Web Server | Nginx on Amazon Linux 2023 |
| Database | Amazon RDS — MySQL 8.0 (`db.t3.micro`) |
| Static Assets | Amazon S3 (`qloan` bucket) |
| Cloud | AWS EC2, VPC, RDS, S3, ALB, Auto Scaling, CloudFormation |
| Region | ap-south-1 (Mumbai) |
| Networking | IGW, Route Tables, Security Groups, 3 Public Subnets across AZs |

---

## 🚀 Deployment

### Step 1 — Deploy infrastructure via CloudFormation

```bash
aws cloudformation deploy \
  --template-file vpc-ec2.yml \
  --stack-name quickloan-stack \
  --region ap-south-1
```

### Step 2 — Set up EC2 instance

```bash
# SSH in
ssh -i AWS.pem ec2-user@<EC2_PUBLIC_IP>

# Configure hostname & install packages
sudo hostnamectl set-hostname appsrv
sudo dnf update -y
sudo dnf install nginx -y
sudo systemctl start nginx && sudo systemctl enable nginx

# Install PHP 8.2
sudo dnf install php8.2 php-fpm php-mysqlnd php-pdo php-mbstring -y
sudo systemctl start php-fpm && sudo systemctl enable php-fpm
sudo systemctl restart nginx

# Install MySQL client (to connect to RDS)
sudo yum install mariadb105 -y
```

### Step 3 — Deploy the application

```bash
sudo cp -r includes/ public/ /usr/share/nginx/html/
sudo chown -R nginx:nginx /usr/share/nginx/html/public
sudo chmod -R 755 /usr/share/nginx/html/public
```

### Step 4 — Initialize the RDS database

```bash
# Connect to RDS and run schema
mysql -h db1.c5u6qi2egbga.ap-south-1.rds.amazonaws.com -P 3306 -u admin -p < init.sql
```

Update `includes/db_connect.php`:

```php
$host = 'db1.c5u6qi2egbga.ap-south-1.rds.amazonaws.com';
$db   = 'quickloan_db';
$user = 'admin';
$pass = 'YOUR_PASSWORD';   // use AWS Secrets Manager in production
```

### Step 5 — Configure Nginx

```bash
sudo cp /usr/share/nginx/html/nginx/quickloan.conf /etc/nginx/conf.d/
sudo nginx -t
sudo systemctl restart nginx
```

### Step 6 — Create AMI & configure Auto Scaling

1. **EC2 Console → Instances** → select your configured instance
2. **Actions → Image and templates → Create image** → name: `quickloan-ami`
3. **Launch Templates** → create using `quickloan-ami`
4. **Auto Scaling Groups** → create `asg1` with Min: 2, Max: 4
5. Attach **ALB `LB1`** as the target group
6. Set scaling policy: scale out at CPU > 70%

---

## 🗄️ Database Schema

```sql
CREATE DATABASE quickloan_db;

USE quickloan_db;

CREATE TABLE applications (
  id         INT AUTO_INCREMENT PRIMARY KEY,
  name       VARCHAR(255) NOT NULL,    -- Applicant full name
  contact    VARCHAR(20)  NOT NULL,    -- Phone number
  email      VARCHAR(255) NOT NULL,    -- Email address
  loan_type  VARCHAR(50)  NOT NULL     -- Home Loan / Gold Loan / Vehicle Loan / Personal Loan
);
```

**Sample data (live in RDS):**

| id | name | contact | email | loan_type |
|---|---|---|---|---|
| 1 | ALAM | 97873876853 | alam@gmail.com | Gold Loan |
| 2 | ajay | 683687538758 | ajay@gmail.com | Personal Loan |
| 3 | ajay | 7838683686 | ajay@gmail.com | Vehicle Loan |

---

## 🔐 Security Notes

- ✅ RDS is in a **private subnet** — no direct internet access
- ✅ RDS Security Group allows **only EC2 SG** on port 3306
- ✅ ALB handles all public-facing HTTP traffic
- ✅ S3 bucket stores only static public assets (images, logo)
- ⚠️ Restrict SSH (port 22) to **your IP only** in production
- ⚠️ Move DB credentials to **AWS Secrets Manager**
- ⚠️ Add **HTTPS** via ACM certificate on ALB for production

---

## 🛣️ Roadmap

- [ ] HTTPS via AWS Certificate Manager + ALB listener
- [ ] Move DB credentials to AWS Secrets Manager
- [ ] User authentication — login and register pages
- [ ] EMI calculator with amortization breakdown
- [ ] Admin dashboard for loan approval / rejection
- [ ] RDS Multi-AZ for high availability
- [ ] CloudWatch alarms for EC2 CPU and RDS connections
- [ ] GitHub Actions CI/CD pipeline

---

## 🤝 Contributing

Pull requests are welcome!

1. Fork the repo
2. Create your branch: `git checkout -b feature/your-feature`
3. Commit: `git commit -m "Add your feature"`
4. Push: `git push origin feature/your-feature`
5. Open a Pull Request

---

## 📄 License

This project is licensed under the [MIT License](LICENSE).

---

<div align="center">
  <p>Made with ❤️ by <a href="https://github.com/purveshchaudhari456">Purvesh Chaudhari</a></p>
  <p>⭐ Star this repo if you found it helpful!</p>
</div>
