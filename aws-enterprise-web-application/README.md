# AWS Enterprise Web Application
### Highly Available, Auto-Scaling Web Infrastructure with Monitoring & Alerting

> **Fresher Cloud Engineer Portfolio Project** | AWS | Linux | Networking | Monitoring

---

## 📌 Project Summary

Designed and deployed a **production-style, fault-tolerant web application** on AWS from scratch — covering custom networking, compute, database, load balancing, auto scaling, monitoring, and alerting. Every layer of the architecture was manually configured to mirror how real cloud teams operate.

---

## 🏗️ Architecture

```
                        ┌─────────────────────────────────────────┐
                        │              prod-vpc (10.0.0.0/16)      │
                        │                                         │
   Internet ──────► Application Load Balancer (web-alb)          │
                        │         (public-subnet-1, public-subnet-2)│
                        │              ↓           ↓              │
                        │    ┌─────────────┐ ┌─────────────┐     │
                        │    │  EC2 (AZ-a) │ │  EC2 (AZ-b) │     │
                        │    │  web-server │ │  ASG launch │     │
                        │    └──────┬──────┘ └──────┬──────┘     │
                        │           └────────┬───────┘            │
                        │                    ↓                    │
                        │          ┌──────────────────┐          │
                        │          │   RDS MySQL       │          │
                        │          │ (private subnets) │          │
                        │          └──────────────────┘          │
                        │                                         │
                        │   CloudWatch ──► SNS ──► Gmail Alert    │
                        └─────────────────────────────────────────┘
```

**Traffic Flow:** `User → ALB → Target Group → Auto Scaling Group (EC2) → Apache`  
**Monitoring Flow:** `CloudWatch (CPU > 70%) → SNS Topic → Email Notification`

---

## ☁️ AWS Services Used

| Category | Service | Purpose |
|---|---|---|
| Networking | VPC, Subnets, IGW, Route Tables | Custom isolated network |
| Security | Security Groups, IAM Roles | Access control |
| Compute | EC2, Auto Scaling Group | Web servers |
| Load Balancing | Application Load Balancer, Target Group | Traffic distribution |
| Database | RDS MySQL | Persistent data layer |
| Monitoring | CloudWatch | CPU & health metrics |
| Alerting | SNS | Email notification on threshold |
| Backup | EBS Snapshots, AMI | Disaster recovery |

---

## 🌐 Networking Design

| Resource | Name | CIDR / Details |
|---|---|---|
| VPC | prod-vpc | 10.0.0.0/16 |
| Public Subnet 1 | public-subnet-1 | 10.0.1.0/24 — ap-south-1a |
| Public Subnet 2 | public-subnet-2 | 10.0.2.0/24 — ap-south-1b |
| Private Subnet 1 | private-subnet-1 | 10.0.3.0/24 — ap-south-1a |
| Private Subnet 2 | private-subnet-2 | 10.0.4.0/24 — ap-south-1b |
| Internet Gateway | prod-igw | Attached to prod-vpc |
| Public Route Table | public-rt | 0.0.0.0/0 → prod-igw |
| Private Route Table | private-rt | Local route only (no internet) |

**Design Decision:** EC2 instances and the ALB sit in **public subnets** for internet access. The RDS database is placed in **private subnets** — it is never directly accessible from the internet, only reachable from the web tier via `db-sg`.

---

## 🔐 Security Groups

### `web-sg` — Applied to EC2 and ALB
| Rule | Port | Source |
|---|---|---|
| HTTP | 80 | 0.0.0.0/0 |
| HTTPS | 443 | 0.0.0.0/0 |
| SSH | 22 | My IP only |

### `db-sg` — Applied to RDS
| Rule | Port | Source |
|---|---|---|
| MySQL/Aurora | 3306 | web-sg only |

**Design Decision:** The database security group only allows inbound traffic **from the web security group**, not from any IP address. This enforces the principle of least privilege at the network layer.

---

## 🖥️ Compute Configuration

| Resource | Value |
|---|---|
| AMI | Amazon Linux 2023 |
| Instance Type | t3.micro |
| Web Server | Apache (httpd 2.4.67) |
| Key Pair | aws.key-mumbai |
| Region | ap-south-1 (Mumbai) |

**Web Server Setup (User Data script used in Launch Template):**
```bash
#!/bin/bash
yum update -y
yum install httpd -y
systemctl start httpd
systemctl enable httpd
echo "Hello from Auto Scaling Server 🚀" > /var/www/html/index.html
```

---

## ⚖️ Load Balancer & Auto Scaling

| Resource | Name | Configuration |
|---|---|---|
| Target Group | web-tg | HTTP / port 80 / health check: `/` |
| Load Balancer | web-alb | Internet-facing, ap-south-1a & 1b |
| Launch Template | web-lt | Amazon Linux 2023, t3.micro, web-sg |
| Auto Scaling Group | web-asg | Min: 2, Desired: 2, Max: 3 |

**ALB DNS:** `web-alb-1495806880.ap-south-1.elb.amazonaws.com`

**Design Decision:** ASG is configured with a **minimum of 2 instances** across two availability zones. If one AZ fails, the other continues serving traffic. This eliminates the single point of failure.

---

## 📊 Monitoring & Alerting

| Resource | Configuration |
|---|---|
| CloudWatch Alarm | Metric: CPUUtilization, Threshold: > 70%, Statistic: Average |
| SNS Topic | web-alerts-v2 (ap-south-1) |
| Notification | Email alert sent when alarm triggers |
| Subscription Status | Confirmed ✅ |

**Alert tested manually via SNS Publish** — email delivery confirmed to Gmail inbox.

---

## 🚨 Troubleshooting Log

Real issues encountered and resolved during the build:

| # | Issue | Root Cause | Fix Applied |
|---|---|---|---|
| 1 | EC2 Instance Connect (browser SSH) failed | Browser-based SSH is unreliable in Mumbai region | Switched to terminal SSH: `ssh -i "key.pem" ec2-user@<IP>` |
| 2 | `chmod` and `sudo` not found in Windows CMD | Linux commands ran in Windows terminal instead of EC2 | Understood the split: Windows CMD = SSH only; EC2 terminal = all setup commands |
| 3 | EC2 not showing in Target Group registration | VPC mismatch during Target Group creation | Verified VPC was set to `prod-vpc` before creating Target Group |
| 4 | SNS email not received | Subscription stuck in `Pending confirmation` + email delivered to Promotions tab | Recreated subscription, confirmed from Gmail Promotions folder |
| 5 | SNS Topic delete error: `Missing required key 'TopicArn'` | AWS console selected `undefined` topic due to partial page load | Refreshed page, clicked into topic directly before deleting |
| 6 | Auto Scaling Group showing only Target Group (no ALB name) | Expected AWS behavior — ASG attaches to TG, not directly to ALB | Confirmed this is correct: ALB → TG → ASG is the proper chain |

---

## ✅ What This Project Demonstrates

| Skill | Evidence |
|---|---|
| AWS Networking | Custom VPC, 4 subnets, IGW, public/private route tables |
| Security Design | Security groups with least-privilege rules, RDS in private subnet |
| High Availability | ALB + ASG across 2 AZs, min 2 instances always running |
| Fault Tolerance | If one EC2 fails, ASG automatically launches a replacement |
| Monitoring | CloudWatch alarm on CPU metric |
| Alerting | SNS email notification, subscription confirmed and tested |
| Linux & Scripting | Apache installed, configured, and automated via user data |
| SSH & Key Management | Connected to EC2 using `.pem` key from Windows terminal |
| Troubleshooting | Independently resolved 6 real issues during build |

---

## 🏆 Resume Bullet Points

```
• Designed and deployed a highly available AWS web application using 
  VPC (public/private subnets), EC2, ALB, Auto Scaling, and RDS MySQL 
  across two availability zones in ap-south-1.

• Configured CloudWatch CPU alarms with SNS email notifications for 
  real-time incident alerting.

• Secured the database tier by placing RDS in private subnets with 
  a security group allowing MySQL access only from the web security group.

• Resolved 6 real infrastructure issues during deployment including 
  SSH connectivity, Target Group VPC mismatch, and SNS subscription failures.
```

---

## 📁 Project Structure

```
aws-enterprise-web-application/
├── README.md
├── architecture/
│   └── architecture-diagram.png
├── screenshots/
│   ├── vpc.png
│   ├── subnets.png
│   ├── security-groups.png
│   ├── ec2-running.png
│   ├── alb-active.png
│   ├── asg-instances.png
│   ├── cloudwatch-alarm.png
│   └── sns-confirmed.png
└── documentation/
    └── project-steps.md
```

---

## 👩‍💻 Author

**Akshaya**  
Cloud Engineer (Fresher) | AWS | Linux | Networking  
Built as part of hands-on cloud engineering portfolio — ap-south-1 (Mumbai)

---

*This project was built entirely manually through the AWS Console — no CloudFormation or Terraform — to develop a deep understanding of each service and how they connect.*
