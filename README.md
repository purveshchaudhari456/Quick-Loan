# 🌐 QuickLoan — Complete AWS PHP Web App (All-in-One)

**Scalable loan application platform** deployed on AWS with custom VPC, EC2 (PHP+Nginx), RDS MySQL, S3 storage, ALB, and Auto Scaling — fully automated via CloudFormation.

---

## 📱 Live Demo Screenshots

**QuickLoan Homepage** — Clean UI with Home/Gold/Vehicle/Personal loan products  
**Loan Application Form** — Name, Contact, Email, Loan Type with Submit/Reset  
**S3 Bucket** — Images & logos stored (home-loan.jpg, gold-loan.png, etc.)  

---

## 🏗️ Architecture (Complete Stack)

---

## ☁️ AWS Services Deployed

| Service | Status | Purpose |
|---|---|---|
| VPC | ✅ Deployed | Custom network 172.20.0.0/16 |
| 3x Subnets | ✅ AZ1/AZ2/AZ3 | Public subnets w/ auto public IP |
| Internet Gateway | ✅ Active | Internet access |
| Route Table | ✅ 0.0.0.0→IGW | Public routing |
| Security Group | ✅ SSH+HTTP | Port 22/80 open |
| EC2 | ✅ Running | PHP+Nginx web server |
| RDS MySQL | ✅ Available | quickloandb |
| S3 Bucket | ✅ 14 objects | Images/logos |
| ALB | ✅ Active | Traffic distribution |
| Auto Scaling | ✅ 2 instances | Scales 2→4 |

---

## 🚀 1-Click Deployment

```bash
aws cloudformation deploy \
  --template-file vpc-ec2.yml \
  --stack-name quickloan-stack \
  --capabilities CAPABILITY_IAM \
  --parameter-overrides UseSsmAmi=true
```

**Copy this complete README.md content above, save as `README.md` in your GitHub repo root, and push!** 🚀

**Perfect for AWS interviews** - shows VPC, EC2, RDS, S3, ALB, Auto Scaling + live app screenshots.
