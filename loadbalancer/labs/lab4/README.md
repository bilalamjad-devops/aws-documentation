

<img width="720" height="590" alt="1_Z-DC516_NPhaBhs3o0VXVg" src="https://github.com/user-attachments/assets/68beede3-9d5f-4ab8-8ed7-b811d71829e0" />

https://medium.com/@aaloktrivedi/leveraging-high-availability-by-creating-an-aws-auto-scaling-group-application-load-balancer-2ea1f31a746



- vpc
- igw
- 2 public subnets
- route table
- 1 sg for both ec2 and asg 80 and 22
- launch template
- target group
- application load balancer
- auto scaling group 





# Lab 2 — AWS Application Load Balancer + Auto Scaling Group using Custom VPC (GUI)

## Goal

Build this architecture:

```txt id="jns3z9"
Internet
   ↓
Application Load Balancer
(public subnets)
   ↓
Target Group
   ↓
Auto Scaling Group
(private subnets)
   ├── EC2 App Server 1
   └── EC2 App Server 2
```

---

# What you will create

✅ Custom VPC

✅ 2 Public subnets



✅ Internet Gateway



✅ Route tables

---
✅ Security Group

✅ Launch Template

✅ Target Group

✅ Application Load Balancer

✅ Auto Scaling Group

---

# Region example

Use:

```txt id="7mkcnv"
ap-south-1 (Mumbai)
```

---

# Step 1 — Create VPC

AWS → VPC

Create VPC

Fill:

| Field     | Value       |
| --------- | ----------- |
| Name      | MyCustomVPC |
| IPv4 CIDR | 10.0.0.0/16 |

Create

---

# Step 2 — Create 2 Public Subnets

Create subnet

### Public-1

| Field | Value       |
| ----- | ----------- |
| Name  | Public-1    |
| AZ    | ap-south-1a |
| CIDR  | 10.0.1.0/24 |

Create

---

### Public-2

| Field | Value       |
| ----- | ----------- |
| Name  | Public-2    |
| AZ    | ap-south-1b |
| CIDR  | 10.0.2.0/24 |

Create



---

# Step 4 — Internet Gateway

VPC → Internet Gateways

Create

Name:

```txt id="fz5kbf"
MyIGW
```

Attach to:

```txt id="3ofh4p"
MyCustomVPC
```

---

# Step 5 — Public Route Table

Create

Name:

```txt id="vj8ntp"
Public-RT
```

Routes

Add:

```txt id="7vv4q4"
0.0.0.0/0 → Internet Gateway
```

Subnet associations:

Attach:

* Public-1
* Public-2



---



---

# Step 9 — Security Group

EC2 → Security Groups

Create

Name:

```txt id="f7o0gp"
WebTraffic-SG
```

VPC:

```txt id="2w5wxy"
MyCustomVPC
```

Inbound:

| Type | Port | Source    |
| ---- | ---: | --------- |
| HTTP |   80 | 0.0.0.0/0 |
| SSH  |   22 | My IP     |

Outbound:

All

Create

---

# Step 10 — Launch Template

EC2 → Launch Templates

Create

| Field    | Value             |
| -------- | ----------------- |
| Name     | MyWebServer-LT    |
| AMI      | Amazon Linux 2023 |
| Type     | t2.micro          |
| Key Pair | your key          |
| SG       | WebTraffic-SG     |

Do NOT select subnet

---

## User Data

Paste:

```bash id="3vgjcx"
#!/bin/bash
yum update -y
yum install -y httpd
systemctl start httpd
systemctl enable httpd

# IMDSv2 secure metadata fetch
TOKEN=$(curl -s -X PUT "http://169.254.169.254/latest/api/token" \
  -H "X-aws-ec2-metadata-token-ttl-seconds: 21600")

INSTANCE_ID=$(curl -s -H "X-aws-ec2-metadata-token: $TOKEN" \
  http://169.254.169.254/latest/meta-data/instance-id)

AVAILABILITY_ZONE=$(curl -s -H "X-aws-ec2-metadata-token: $TOKEN" \
  http://169.254.169.254/latest/meta-data/placement/availability-zone)

cat > /var/www/html/index.html <<EOF
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>ALB + ASG Lab</title>
    <style>
        body { font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; background-color: #f4f6f9; margin: 0; padding: 20px; color: #333; }
        .container { max-width: 800px; margin: 40px auto; background: white; padding: 30px; border-radius: 8px; box-shadow: 0 4px 12px rgba(0,0,0,0.1); }
        .header { display: flex; align-items: center; border-bottom: 2px solid #f0f2f5; padding-bottom: 15px; margin-bottom: 25px; }
        .logo { font-weight: bold; color: #ff9900; font-size: 24px; margin-right: 20px; }
        table { width: 100%; border-collapse: collapse; margin-top: 20px; }
        th, td { border: 1px solid #e0e0e0; padding: 12px 15px; text-align: left; font-size: 16px; }
        th { background-color: #f8f9fa; font-weight: 600; color: #555; }
        td { font-family: 'Courier New', Courier, monospace; font-weight: bold; color: #444; }
    </style>
</head>
<body>
<div class="container">
    <div class="header">
        <div class="logo">aws</div>
    </div>
    <table>
        <thead>
            <tr><th>Meta-Data</th><th>Value</th></tr>
        </thead>
        <tbody>
            <tr><td>Instance ID</td><td>${INSTANCE_ID}</td></tr>
            <tr><td>Availability Zone</td><td>${AVAILABILITY_ZONE}</td></tr>
        </tbody>
    </table>
</div>
</body>
</html>
EOF
```

Create

---

# Step 11 — Target Group

EC2 → Target Groups

Create

| Field        | Value         |
| ------------ | ------------- |
| Name         | WebServers-TG |
| Type         | Instances     |
| Port         | 80            |
| VPC          | MyCustomVPC   |
| Health Check | /             |

Create

Skip register

---

# Step 12 — Application Load Balancer

EC2 → Load Balancers

Create

Application LB

Fill:

| Field  | Value           |
| ------ | --------------- |
| Name   | MyALB           |
| Scheme | Internet-facing |
| VPC    | MyCustomVPC     |

Select public subnets:

✅ Public-1

✅ Public-2

Security Group:

```txt id="cgjm1h"
WebTraffic-SG
```

Listener:

HTTP 80

Forward:

```txt id="2b8a5w"
WebServers-TG
```

Create

---

# Step 13 — Auto Scaling Group

EC2 → Auto Scaling Groups

Create

| Field           | Value          |
| --------------- | -------------- |
| Name            | MyASG          |
| Launch Template | MyWebServer-LT |

Next

VPC:

```txt id="h2h4nt"
MyCustomVPC
```

Subnets:

✅ Public-1

✅ Public-2

Next

Attach to load balancer

Select:

```txt id="nmq1h2"
WebServers-TG
```

Health checks:

Enable ELB

Capacity:

| Desired | 2 |
| ------- | - |
| Min     | 2 |
| Max     | 4 |

Create

---

# Step 14 — Wait

2–5 minutes

Check:

EC2 → Instances

You should see:

```txt id="kucb0o"
2 running
```

Private IP only

---

# Step 15 — Test

EC2 → Load Balancers

Copy DNS

Open browser

Example:

```txt id="xjefb1"
http://MyALB-xxxxx.ap-south-1.elb.amazonaws.com
```

Page:

```txt id="yq6h4v"
AWS Custom VPC Lab
Instance ID: i-xxxx
AZ: ap-south-1a
```

Refresh

Now:

```txt id="93qx11"
Instance ID changes
AZ changes
```

That proves:

✅ custom VPC works
✅ igw works
✅ ALB public
✅ ASG public
✅ traffic balancing works

---



# Cleanup

Delete in order:

1. ASG
2. EC2
3. ALB
4. Target Group
5. Launch Template
8. Route Tables
9. Subnets
10. IGW
11. VPC

---




