# EC2 Web Server with VPC & Security Groups

> A production-style AWS project: deploying a live Apache web server on EC2 inside a custom VPC with IAM-controlled Security Groups and a static Elastic IP.

---

## Live Demo

> **http://YOUR_ELASTIC_IP** ← Replace this with your actual Elastic IP after deployment

---

## Project Overview

This project demonstrates how to build and secure a web server on AWS from scratch — without using any default VPCs or pre-configured resources. Every component (VPC, subnet, Internet Gateway, route table, security group) was created and configured manually to follow real-world cloud engineering practices.

---

## Architecture Diagram

```
                        INTERNET
                           |
                    [ Internet Gateway ]
                           |
              +------------+------------+
              |        my-web-vpc       |
              |      (10.0.0.0/16)      |
              |                         |
              |  +-------------------+  |
              |  |  my-public-subnet |  |
              |  |   (10.0.1.0/24)   |  |
              |  |                   |  |
              |  |  +-----------+    |  |
              |  |  |  EC2 t2   |    |  |
              |  |  |  .micro   |    |  |
              |  |  | Amazon    |    |  |
              |  |  | Linux     |    |  |
              |  |  | Apache    |    |  |
              |  |  +-----------+    |  |
              |  |        |          |  |
              |  |  [Security Group] |  |
              |  |  Port 80  - open  |  |
              |  |  Port 443 - open  |  |
              |  |  Port 22  - My IP |  |
              |  +-------------------+  |
              +-------------------------+
                           |
                      [Elastic IP]
                    (Static Public IP)
```

---

## AWS Services Used

| Service | Purpose |
|---|---|
| EC2 (t2.micro) | Virtual server to host the web application |
| VPC | Isolated private network for all resources |
| Public Subnet | Network segment with internet access |
| Internet Gateway | Allows traffic between VPC and the internet |
| Route Table | Directs internet-bound traffic to the IGW |
| Security Groups | Firewall — controls inbound/outbound traffic |
| IAM | Least-privilege access for SSH (My IP only) |
| Elastic IP | Static public IP so the server URL never changes |
| Amazon Linux 2023 | OS — AWS-optimised Linux for EC2 |
| Apache (httpd) | Web server software to serve the HTML page |

---

## Project Structure

```
ec2-web-server/
├── index.html          # Web page served by Apache
├── setup.sh            # Bash script to install and configure Apache
├── .gitignore          # Excludes .pem keys and OS files
├── docs/
│   └── vpc-config.txt  # VPC, subnet, security group configuration notes
└── screenshots/
    └── live-site.png   # Screenshot of live website (add after deployment)
```

---

## Step-by-Step Setup

### Prerequisites
- AWS account (Free Tier is sufficient)
- AWS Management Console access
- A terminal with SSH support (Linux/Mac terminal or Git Bash on Windows)

---

### Step 1 — Create VPC and Networking

1. Go to **AWS Console → VPC → Your VPCs → Create VPC**
2. Set Name: `my-web-vpc`, CIDR: `10.0.0.0/16`
3. Go to **Subnets → Create Subnet** inside `my-web-vpc`
   - Name: `my-public-subnet`, AZ: `ap-south-1a`, CIDR: `10.0.1.0/24`
4. Go to **Internet Gateways → Create** → Name: `my-igw`
   - Actions → Attach to VPC → select `my-web-vpc`
5. Go to **Route Tables** → select the route table for your VPC
   - Routes → Edit → Add route: `0.0.0.0/0` → Target: `my-igw`

---

### Step 2 — Create Security Group

1. Go to **VPC → Security Groups → Create**
2. Name: `my-web-sg`, VPC: `my-web-vpc`
3. Add Inbound Rules:

| Type  | Protocol | Port | Source    |
|-------|----------|------|-----------|
| HTTP  | TCP      | 80   | 0.0.0.0/0 |
| HTTPS | TCP      | 443  | 0.0.0.0/0 |
| SSH   | TCP      | 22   | My IP     |

> **Security note:** SSH is restricted to "My IP" only — this follows the principle of least privilege and prevents brute-force attacks from unknown IPs.

---

### Step 3 — Launch EC2 Instance

1. Go to **EC2 → Launch Instance**
2. Use these settings:
   - AMI: `Amazon Linux 2023`
   - Instance type: `t2.micro` (Free Tier)
   - Key pair: Create new → download `my-key.pem` and save it safely
   - Network: `my-web-vpc`, Subnet: `my-public-subnet`
   - Security Group: `my-web-sg`
3. After launch, go to **Elastic IPs → Allocate → Associate** with your instance

---

### Step 4 — SSH into EC2 and Deploy Web Server

```bash
# Set correct permissions on your key file
chmod 400 my-key.pem

# SSH into the EC2 instance
ssh -i my-key.pem ec2-user@YOUR_ELASTIC_IP

# Once inside — run the setup script
sudo bash setup.sh
```

Or manually:

```bash
sudo yum update -y
sudo yum install httpd -y
sudo systemctl start httpd
sudo systemctl enable httpd
sudo cp index.html /var/www/html/index.html
```

---

### Step 5 — Verify Live Website

Open your browser and go to:

```
http://YOUR_ELASTIC_IP
```

You should see the live web page hosted on your AWS EC2 instance.

---

## Key Concepts Demonstrated

- **Custom VPC** — instead of using AWS default VPC, built from scratch with manual CIDR planning
- **Least Privilege Security** — SSH access restricted to a specific IP, not open to the world
- **Infrastructure as Documentation** — all configs saved in `docs/vpc-config.txt` for reproducibility
- **Static IP with Elastic IP** — ensures the server URL remains stable across restarts
- **Service auto-start** — `systemctl enable httpd` ensures Apache survives reboots

---

## How to Stop the EC2 (Important — avoid charges)

```
AWS Console → EC2 → Instances → Select instance → Instance State → Stop
```

> Always stop your EC2 when not in use. Free Tier gives 750 hours/month — stopping prevents accidental charges.

---

## Author

**Edwin Jerald S**
AWS Cloud Engineer | DevOps Enthusiast | Linux Administrator
- Email: edwingerald29@gmail.com
- Location: Chennai, India
- Certifications: AWS Certified | MongoDB Certified | MySQL Certified
