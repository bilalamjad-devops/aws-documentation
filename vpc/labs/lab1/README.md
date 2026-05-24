

# 🧪 Lab 1: Create Basic VPC (Public + Private)
Awesome—your cousin is guiding you the right way.

And your idea is solid: before doing the full Terraform architect-level project, first learn each AWS networking piece manually in GUI.

That makes Terraform *much* easier later because you’ll already understand what each resource does.

Here’s a hands-on AWS Console lab focused exactly on:

✅ Create VPC
✅ Create Public Subnet
✅ Create Private Subnet
✅ Launch public EC2
✅ Launch private EC2
✅ SSH into public server
✅ Jump from public → private server (bastion host)

---

# Project 2 Lab (GUI First)

# Scalable Multi-Tier Web App on AWS

### Goal

Build AWS networking manually:

```text
VPC
│
├── Public Subnet
│    └── Bastion Host (EC2 with public IP)
│
└── Private Subnet
     └── App Server EC2 (No public IP)
```

Then connect:

```text
Laptop → Public EC2 → Private EC2
```

This is real-world architecture.

---

# Architecture

```text
Internet
   │
   ▼
Internet Gateway
   │
   ▼
VPC (10.0.0.0/16)
│
├── Public Subnet (10.0.1.0/24)
│      └── Bastion Host
│
└── Private Subnet (10.0.2.0/24)
       └── Private EC2
```

---

# Part 1 — Create VPC

AWS Console → VPC

Click:

**Create VPC**

Choose:

```text
VPC only
```

Name:

```text
MyProject-VPC
```

IPv4 CIDR:

```text
10.0.0.0/16
```

Create

Done.

---

# Part 2 — Create Internet Gateway

VPC → Internet Gateways

Create:

```text
MyProject-IGW
```

After create:

Attach to VPC

Choose:

```text
MyProject-VPC
```

Save

Now internet can reach VPC.

---

# Part 3 — Public Subnet

VPC → Subnets

Create

Values:

Name:

```text
Public-Subnet-1
```

VPC:

```text
MyProject-VPC
```

AZ:

```text
ap-south-1a
```

CIDR:

```text
10.0.1.0/24
```

Create

---

Enable auto public IP:

Select subnet

Actions

Edit subnet settings

Enable:

✅ Auto-assign public IPv4

Save

---

# Part 4 — Private Subnet

Create subnet

Name:

```text
Private-Subnet-1
```

AZ:

```text
ap-south-1a
```

CIDR:

```text
10.0.2.0/24
```

Create

Important:

❌ do NOT enable auto public IP

---

# Part 5 — Route Tables

Need 2 route tables

---

## Public Route Table

Route Tables

Create

Name:

```text
Public-RT
```

VPC:

```text
MyProject-VPC
```

Create

Routes tab

Add route

Destination:

```text
0.0.0.0/0
```

Target:

```text
Internet Gateway
```

Choose:

```text
MyProject-IGW
```

Save

Subnet associations

Associate:

```text
Public-Subnet-1
```

---

## Private Route Table

Create

Name:

```text
Private-RT
```

Associate with:

```text
Private-Subnet-1
```

No internet route

Save

---

# Part 6 — Security Groups

Create 2

---

## Bastion SG

EC2 → Security Groups

Create

Name:

```text
Bastion-SG
```

Inbound:

SSH

```text
22
```

Source:

```text
My IP
```

Save

---

## Private Server SG

Create

Name:

```text
Private-SG
```

Inbound:

SSH

Port:

```text
22
```

Source:

Choose:

```text
Bastion-SG
```

Meaning:

Only bastion can access private server.

Very important.

---

# Part 7 — Key Pair

EC2 → Key pairs

Create

Name:

```text
aws-lab-key
```

PEM

Download

Save file carefully.

---

# Part 8 — Launch Bastion Host

Launch EC2

Name:

```text
Bastion-Host
```

AMI:

```text
Amazon Linux 2023
```

Type:

```text
t2.micro
```

Key:

```text
aws-lab-key
```

Network:

```text
MyProject-VPC
```

Subnet:

```text
Public-Subnet-1
```

Auto assign IP:

```text
Enable
```

Security group:

```text
Bastion-SG
```

Launch

Copy public IP.

---

# Part 9 — Launch Private EC2

Launch

Name:

```text
Private-App-Server
```

AMI:

```text
Amazon Linux
```

Type:

```text
t2.micro
```

Key:

```text
aws-lab-key
```

VPC:

```text
MyProject-VPC
```

Subnet:

```text
Private-Subnet-1
```

Public IP:

```text
Disable
```

Security group:

```text
Private-SG
```

Launch

Copy private IP

Example:

```text
10.0.2.50
```

---

# Part 10 — SSH into Bastion

Terminal

```bash
chmod 400 aws-lab-key.pem
```

Connect:

```bash
ssh -i aws-lab-key.pem ec2-user@PUBLIC_IP
```

Example:

```bash
ssh -i aws-lab-key.pem ec2-user@13.233.x.x
```

Connected.

---

# Part 11 — Copy key inside bastion

On laptop:

```bash
scp -i aws-lab-key.pem aws-lab-key.pem ec2-user@PUBLIC_IP:/home/ec2-user/
```

Now SSH to bastion again

Inside bastion:

```bash
chmod 400 aws-lab-key.pem
```

---

# Part 12 — Jump to private server

Inside bastion:

```bash
ssh -i aws-lab-key.pem ec2-user@10.0.2.50
```

Success 🎉

Flow:

```text
Laptop
   ↓
Bastion Host
   ↓
Private Server
```

That’s exactly how companies manage private servers.

---

# Test

Inside private server:

```bash
hostname
```

Check private IP:

```bash
ip a
```

Ping bastion:

```bash
ping 10.0.1.x
```

---

# Cost Optimization

Use:

```text
t2.micro
```

Delete after lab:

* EC2
* VPC
* IGW
* Route tables

RDS later costs more—skip for now.

---

# What next after this

After you finish this GUI lab:

Step 2:

Learn:

✅ Load Balancer

Step 3:

Learn:

✅ Auto Scaling Group

Step 4:

Learn:

✅ RDS private subnet

Step 5:

Convert all into Terraform

Then final project:

# Scalable Multi-Tier Web App on AWS

```text
Terraform
→ VPC
→ Subnets
→ Bastion
→ ALB
→ ASG
→ RDS
```

Perfect architect-level project.

And once you finish GUI subnet/jump-host lab, message me and I’ll give you the next lab:

**“Create ALB + Auto Scaling Group using AWS Console”**
