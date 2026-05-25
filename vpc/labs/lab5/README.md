
# How to create VPC with 2 public subnets, IGW, Route table, ALB and ASG and 2 SGs






### 📋 Aapki Final Corrected List (The Perfect Checklist)

Agar aap apni copy par checklist bana rahe hain, toh yeh aapki final full list honi chahiye:

* **VPC** (Isolated Network Room)
* **2 Public Subnets** (Alag-alag Availability Zones mein, e.g., `ap-south-1a` aur `ap-south-1b`)
* **Internet Gateway (IGW)** (VPC ka main entrance gate)
* **Route Table** (Signboard jo public subnets ko IGW ka rasta dikhaye)
* **2 Security Groups:**
   * Aik ALB ke liye (Open Port 80 from everywhere `0.0.0.0/0`)
   * Aik EC2 ke liye (Open Port 80 only from ALB Security Group)


* **Target Group (TG)** (Protocol HTTP, Port 80)
* **Launch Template (LT)** (Select Ubuntu, `t2.micro`, EC2 Security Group, aur advanced details mein **User Data Code**)
* **Application Load Balancer (ALB)** (Attach to 2 Public Subnets, use ALB Security Group, aur forward traffic to Target Group)
* **Auto Scaling Group (ASG)** (Attach to the same 2 Subnets, use Launch Template, aur target selection mein select your Target Group)





### Sawaal: Best practice ke mutabiq mujhe total kitne Security Groups (SGs) banane chahiye?

Is lab ke liye (VPC + Public Subnets + ALB + ASG/EC2) aapko **Total 2 Security Groups** banane chahiye:

1. **ALB Security Group (The Public Firewall):**
* **Inbound Rule:** Poori duniya se internet traffic allow karega (`Port 80 / HTTP` from `0.0.0.0/0`).
* **Maqsad:** Takay jab koi browser mein aapka URL hit kare, toh Load Balancer request receive kar sake.


2. **EC2/ASG Security Group (The Private Firewall):**
* **Inbound Rule:** Yeh poori duniya ke liye block hoga! Yeh sirf aur sirf **ALB Security Group ki ID** se traffic allow karega (`Port 80 / HTTP` from `sg-alb-id`).
* *Optional:* Agar aapko public server se SSH karna hai, toh aap isme `Port 22` open kar sakte hain.
* **Maqsad:** Takay koi bhi hacker direct aapke servers par attack na kar sake. Traffic hamesha Load Balancer se ho kar hi guzre.


