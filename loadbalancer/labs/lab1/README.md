# AWS Lab: Application Load Balancer + Auto Scaling Group

## What We Are Building & How It Works

Before touching the AWS console, let me explain exactly what you will build and why.

**You will create:**
1. A **Security Group** — acts as a firewall (one group, shared by both ALB and EC2 instances)
2. A **Launch Template** — a blueprint that tells AWS how to create each EC2 instance
3. A **Target Group** — a list of EC2 instances that the Load Balancer will send traffic to
4. An **Application Load Balancer (ALB)** — the single entry point that users hit in their browser
5. An **Auto Scaling Group (ASG)** — automatically creates/destroys EC2 instances based on demand

**The request flow when a user opens the Load Balancer URL in a browser:**

```
User's Browser
      │
      ▼
Application Load Balancer   ← only this has a public URL
      │
      ├──▶ EC2 Instance in AZ-1a  (shows its own Instance ID + AZ)
      │
      └──▶ EC2 Instance in AZ-1b  (shows its own Instance ID + AZ)
           (managed by Auto Scaling Group)
```

Every time you refresh the browser, the ALB sends your request to a **different EC2 instance**, so you see a different Instance ID and Availability Zone. That is the whole magic of this lab.

---

## Step 1 — Create the Security Group

A Security Group is a virtual firewall. We need to allow HTTP traffic (port 80) so the browser can reach our web pages, and SSH (port 22) so we can log into instances if needed.

> **Why one Security Group for everything?** The ALB receives traffic from the internet on port 80, and the EC2 instances also serve traffic on port 80. Sharing one group keeps the lab simple.

1. Open the AWS Console and go to **EC2**
2. In the left sidebar, scroll down and click **Security Groups**
3. Click the orange **Create security group** button
4. Fill in the form:

| Field | Value |
|-------|-------|
| Security group name | `WebTraffic-SG` |
| Description | Allow HTTP from internet and SSH for admin |
| VPC | Select your default VPC |

5. Under **Inbound rules**, click **Add rule** twice and add:

| Type | Protocol | Port | Source | Why |
|------|----------|------|--------|-----|
| HTTP | TCP | 80 | 0.0.0.0/0 | Let browser traffic reach ALB and EC2 |
| SSH | TCP | 22 | My IP | Let you log in for troubleshooting |

6. Leave **Outbound rules** as default (allow all)
7. Click **Create security group**

✅ You now have a firewall rule ready. Name `WebTraffic-SG` makes it clear — this group allows web traffic.

---

## Step 2 — Create the Launch Template

A Launch Template is like a cookie cutter. Every time the Auto Scaling Group needs a new EC2 instance, it uses this template to stamp one out — same OS, same instance type, same startup script every time.

The startup script (called **User Data**) runs automatically when an EC2 instance boots. It will install Apache web server and create an HTML page that shows that instance's own ID and Availability Zone.

1. In the EC2 left sidebar, click **Launch Templates**
2. Click **Create launch template**
3. Fill in the top section:

| Field | Value |
|-------|-------|
| Launch template name | `MyWebServer-LT` |
| Template version description | Version 1 — Apache with metadata page |
| ✅ Check the box | "Provide guidance to help me set up a template that I can use with EC2 Auto Scaling" |

4. Under **Application and OS Images (AMI)**, click **Quick Start** → select **Amazon Linux** → choose **Amazon Linux 2023 AMI** (the one that says Free tier eligible)

5. Under **Instance type**, select **t2.micro** (Free tier eligible)

6. Under **Key pair**, select any existing key pair you have. If you have none, click **Create new key pair**, name it `MyKeyPair`, keep RSA + .pem, and download it.

7. Under **Network settings**:
   - Do NOT set a subnet here (the ASG will handle subnet placement)
   - Under **Firewall (security groups)**, select **Select existing security group**
   - Choose `WebTraffic-SG` that you created in Step 1

8. Scroll all the way down to **Advanced details**. Click to expand it.

9. Scroll to the very bottom of Advanced details and find the **User data** text box. Paste this entire script:

```bash
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

10. Click **Create launch template**

✅ The Launch Template `MyWebServer-LT` is ready. Every EC2 instance born from this template will automatically install Apache and serve a webpage showing its own identity.

---

## Step 3 — Create a Target Group

A Target Group is simply a **list of EC2 instances** that the Load Balancer will forward traffic to. The ALB does not talk to instances directly — it talks to a Target Group, and the Target Group holds the actual instances.

The ALB also uses the Target Group to run **health checks** — it pings each instance every few seconds on port 80. If an instance stops responding, the ALB automatically removes it from rotation until it recovers.

1. In the EC2 left sidebar, scroll down and click **Target Groups**
2. Click **Create target group**
3. Under **Choose a target type**, select **Instances**
4. Fill in:

| Field | Value |
|-------|-------|
| Target group name | `WebServers-TG` |
| Protocol | HTTP |
| Port | 80 |
| VPC | Select your default VPC |
| Health check protocol | HTTP |
| Health check path | `/` |

5. Leave everything else as default
6. Click **Next**
7. On the **Register targets** page, do NOT add any instances manually. The Auto Scaling Group will register instances automatically.
8. Click **Create target group**

✅ Target Group `WebServers-TG` is ready and empty for now. The ASG will fill it.

---

## Step 4 — Create the Application Load Balancer

The Application Load Balancer is the **single public URL** you will share with users. It sits in front of all your EC2 instances and distributes incoming requests among them in round-robin order.

1. In the EC2 left sidebar, click **Load Balancers**
2. Click **Create load balancer**
3. Under the three options shown, click **Create** under **Application Load Balancer**
4. Fill in the **Basic configuration**:

| Field | Value |
|-------|-------|
| Load balancer name | `MyALB` |
| Scheme | Internet-facing |
| IP address type | IPv4 |

5. Under **Network mapping**:
   - VPC: Select your **default VPC**
   - Under **Mappings**, you will see a list of Availability Zones. **Check at least 2 boxes** (e.g. us-east-1a and us-east-1b). This is important — the ALB must span multiple AZs to distribute traffic across them.
   - For each checked AZ, select a **public subnet** from the dropdown that appears

6. Under **Security groups**:
   - Remove the default security group if pre-selected
   - Select `WebTraffic-SG`

7. Under **Listeners and routing**:
   - Protocol: **HTTP** | Port: **80**
   - Default action: **Forward to** → select `WebServers-TG`

8. Click **Create load balancer**

It will take about 2–3 minutes to become **Active**. You will see the state say "provisioning" — that is normal.

✅ The Load Balancer `MyALB` is created. Copy its **DNS name** from the details page (it looks like `MyALB-123456789.us-east-1.elb.amazonaws.com`) — you will use this in the browser at the end.

---

## Step 5 — Create the Auto Scaling Group

The Auto Scaling Group (ASG) is the brain that **creates and manages your EC2 instances**. You tell it: use this Launch Template, launch in these subnets, and keep at least 2 instances running. It does the rest — including registering those instances into the Target Group automatically.

1. In the EC2 left sidebar, click **Auto Scaling Groups**
2. Click **Create Auto Scaling group**

### Page 1 — Name and Launch Template
| Field | Value |
|-------|-------|
| Auto Scaling group name | `MyASG` |
| Launch template | `MyWebServer-LT` |
| Version | Latest (1) |

Click **Next**

### Page 2 — Network (Instance Launch Options)
- VPC: Select your **default VPC**
- Availability Zones and subnets: Select the **same 2 subnets** you selected for the ALB (one per AZ)

Click **Next**

### Page 3 — Load Balancing and Health Checks
This is where you connect the ASG to your ALB so instances are registered automatically.

- Under **Load balancing**, select **Attach to an existing load balancer**
- Select **Choose from your load balancer target groups**
- From the dropdown, select `WebServers-TG`
- Under **Health checks**, check ✅ **Turn on Elastic Load Balancing health checks**

> This means: if the ALB health check fails for an instance, the ASG will terminate it and launch a fresh replacement automatically.

Click **Next**

### Page 4 — Group Size and Scaling
| Field | Value | Why |
|-------|-------|-----|
| Desired capacity | 2 | Start with 2 instances |
| Minimum capacity | 2 | Never go below 2 |
| Maximum capacity | 4 | Can scale up to 4 under load |

- Under **Automatic scaling**, select **Target tracking scaling policy**
- Metric type: **Average CPU utilization**
- Target value: **50** (scale out when average CPU goes above 50%)

Click **Next → Next → Next → Create Auto Scaling group**

✅ The ASG `MyASG` is now created. It will immediately launch 2 EC2 instances in the background using your `MyWebServer-LT` template and register them into `WebServers-TG`.

---

## Step 6 — Watch Everything Come Alive

### Check your EC2 Instances
1. Go to **EC2 → Instances**
2. You should see **2 new instances** launching with names from your ASG. Wait until their **Instance State** shows `Running` and **Status check** shows `2/2 checks passed` (takes about 2–3 minutes)

### Check your Target Group
1. Go to **EC2 → Target Groups → WebServers-TG**
2. Click the **Targets** tab
3. You should see both instances listed as **healthy** (green)
4. If they show **initial** or **unhealthy**, wait another minute and refresh — the User Data script needs time to install Apache

### Open the Load Balancer in your Browser
1. Go to **EC2 → Load Balancers → MyALB**
2. Copy the **DNS name** from the details panel at the bottom
3. Paste it into your browser and hit Enter
4. You should see the AWS-styled table showing an Instance ID and Availability Zone
5. **Keep pressing F5 to refresh** — you will see the Instance ID and AZ switch between your two instances as the ALB alternates between them

---

## Full Architecture Summary

```
Internet
    │
    ▼
MyALB  (Application Load Balancer — public DNS)
    │   uses WebTraffic-SG (port 80 open)
    │
    ▼
WebServers-TG  (Target Group — health checks every 30s)
    │
    ├──▶ EC2 Instance #1 (us-east-1a) ← created by MyASG using MyWebServer-LT
    │
    └──▶ EC2 Instance #2 (us-east-1b) ← created by MyASG using MyWebServer-LT

MyASG watches CPU — if > 50%, launches more instances (up to 4)
                   if < 50%, terminates extra instances (down to 2)
```

---

## Cleanup — Delete Everything (to avoid AWS charges)

Delete in this exact order — if you delete out of order, AWS will give dependency errors.

1. **Auto Scaling Group** → EC2 → Auto Scaling Groups → select `MyASG` → Actions → Delete
2. Wait for ASG to terminate both EC2 instances (check EC2 → Instances)
3. **Load Balancer** → EC2 → Load Balancers → select `MyALB` → Actions → Delete
4. **Target Group** → EC2 → Target Groups → select `WebServers-TG` → Actions → Delete
5. **Launch Template** → EC2 → Launch Templates → select `MyWebServer-LT` → Actions → Delete
6. **Security Group** → EC2 → Security Groups → select `WebTraffic-SG` → Actions → Delete security group
