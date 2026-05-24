# Lab 1 (Combined)

# AWS VPC Networking Lab: Public & Private Subnets with Bastion Host and NAT Gateway



### Goal

Build this architecture manually in AWS Console:

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
│   ├── Bastion Host (public IP)
│   └── NAT Gateway
│
└── Private Subnet (10.0.2.0/24)
    └── Private EC2 (no public IP)
```

Then test:

```text
Laptop → Bastion → Private EC2
```

and

```text
Private EC2 → Internet through NAT
```

This matches real production networking.

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

CIDR:

```text
10.0.0.0/16
```

Create

Done.

---

# Part 2 — Internet Gateway

VPC → Internet Gateways

Create

Name:

```text
MyProject-IGW
```

Create

Then:

Attach to VPC

Choose:

```text
MyProject-VPC
```

Save

---

# Part 3 — Public Subnet

VPC → Subnets

Create subnet

Name:

```text
Public-Subnet-1
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

Enable public IP:

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

Need 2

---

## Public Route Table

Create

Name:

```text
Public-RT
```

Routes

Add:

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

Subnet associations:

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

Associate:

```text
Private-Subnet-1
```

Leave empty for now

Save

---

# Part 6 — NAT Gateway

Needed so private EC2 can access internet

---

## Allocate Elastic IP

VPC

Elastic IPs

Allocate

Save

---

## Create NAT

VPC → NAT Gateways

Create

Name:

```text
MyProject-NAT
```

Subnet:

```text
Public-Subnet-1
```

Elastic IP:

Select allocated IP

Create

Wait until:

```text
Available
```

---

## Update Private Route Table

Open:

```text
Private-RT
```

Routes

Add:

Destination:

```text
0.0.0.0/0
```

Target:

```text
NAT Gateway
```

Choose:

```text
MyProject-NAT
```

Save

Now private subnet gets outbound internet.

---

# Part 7 — Security Groups

Create 2

---

## Bastion-SG

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

## Private-SG

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

Save

Only bastion can SSH.

---

# Part 8 — Key Pair

EC2 → Key pairs

Create

Name:

```text
aws-lab-key
```

Type:

```text
PEM
```

Download

Keep safe

---

# Part 9 — Launch Bastion Host

EC2 → Launch instance

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

Auto public IP:

```text
Enable
```

Security group:

```text
Bastion-SG
```

Launch

Copy public IP

Example:

```text
13.xx.xx.xx
```

---

# Part 10 — Launch Private EC2

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

Network:

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

# Part 11 — SSH to Bastion

Your terminal:

```bash
chmod 400 aws-lab-key.pem
```

Connect:

```bash
ssh -i aws-lab-key.pem ec2-user@PUBLIC_IP
```

Example:

```bash
ssh -i aws-lab-key.pem ec2-user@13.xx.xx.xx
```

---

# Part 12 — Copy key to bastion

From laptop:

```bash
scp -i aws-lab-key.pem aws-lab-key.pem ec2-user@PUBLIC_IP:/home/ec2-user/
```

SSH again

Inside bastion:

```bash
chmod 400 aws-lab-key.pem
```

---

# Part 13 — Jump to private EC2

Inside bastion:

```bash
ssh -i aws-lab-key.pem ec2-user@10.0.2.50
```

Success 🎉

Flow:

```text
Laptop
   ↓
Bastion
   ↓
Private EC2
```

---

# Part 14 — Test NAT

Inside private EC2 run:

Check internet:

```bash
ping google.com
```

or

```bash
sudo yum update -y
```

If works:

✅ NAT working

Private EC2 has internet without public IP

Exactly how production works.

---

# Final architecture

```text
Internet
   │
IGW
   │
Public Subnet
├── Bastion Host
└── NAT Gateway
      │
      ▼
Private Subnet
└── Private EC2
```

---

# Cost reminder

Use:

```text
t2.micro
```

Delete after practice:

✅ EC2

✅ NAT Gateway

✅ Elastic IP

✅ Route tables

✅ VPC

Important:

⚠ NAT Gateway costs money

Delete it after lab

---

----

---

Thank you — glad it helped.

And very good question. That part is important to understand before moving to ALB/ASG.

---

# 1) Why we created **2 Security Groups?**

Short answer:

Because each server has a different job.

AWS security groups are like **firewall rules attached to each server**.

We made two so access stays controlled.

---

## Security Group 1 — Bastion-SG

Attached to:

```text
Bastion Host (public EC2)
```

Rule:

```text
Inbound:
SSH (22)
Source = My IP
```

Meaning:

```text
Your laptop → Bastion = allowed
Anyone else = blocked
```

Example:

```text
Laptop (your IP)
      ↓ SSH
Bastion Host
```

Why?

Because bastion is the only machine exposed to internet.

---

## Security Group 2 — Private-SG

Attached to:

```text
Private EC2
```

Rule:

```text
Inbound:
SSH (22)
Source = Bastion-SG
```

Meaning:

```text
Bastion → Private EC2 = allowed
Internet → Private EC2 = blocked
Laptop → Private EC2 = blocked
```

Example:

```text
Laptop
   ↓
Bastion
   ↓
Private EC2
```

---

## Why not use ONE security group?

You *can* technically.

But it becomes messy and less secure.

Example if same SG:

```text
SSH from internet
SSH between servers
all mixed together
```

Then private EC2 may accidentally become reachable.

Production teams separate them.

That gives:

✅ better security

✅ easier troubleshooting

✅ cleaner architecture

✅ follows AWS best practice

---

## Real world example

Think of office building:

Bastion host:

```text
Main gate guard
```

Private server:

```text
Server room inside building
```

Rules:

```text
You can enter main gate
Only guard can enter server room
```

That’s exactly why 2 SGs.

---

