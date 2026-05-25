
https://www.linkedin.com/pulse/set-up-amazon-rds-database-bilal-amjad-axnef/?trackingId=Lk3GHKWSQhmjsg9xf5Otjg%3D%3D


https://www.canva.com/design/DAGUgs2gVoo/tGSJIKnj2QqAntUXdzmgFQ/edit

# 🧪 Lab 1: Create RDS and Connect from EC2

📁 `rds/labs/lab1-rds-ec2-connection/README.md`

## 🎯 Objective

Create an RDS MySQL database and connect it from an EC2 instance.

---

## 🏗️ Architecture

```id="s6rwcc"
EC2 (Public Subnet)
        ↓
RDS (Private Subnet)
```

---

## 🚀 Steps

### Step 1: Create RDS

1. Go to **RDS → Create database**
2. Choose:

   * Engine: MySQL
   * Template: Free tier
3. Set:

   * DB name: `mydb`
   * Username: `admin`
   * Password: `StrongPassword123`
4. Disable public access
5. Create database

---

### Step 2: Configure Security Group

* Allow **MySQL (3306)**
* Source: EC2 security group

---

### Step 3: Launch EC2

Install MySQL client:

```bash id="k7qg9g"
sudo apt update
sudo apt install mysql-client -y
```

---

### Step 4: Connect to RDS

```bash id="2v6fxs"
mysql -h <rds-endpoint> -u admin -p
```

---

## ✅ Expected Outcome

* EC2 connects successfully to RDS
* You can run SQL queries

---

## 🔐 Learning

👉 RDS is never accessed directly from internet
👉 Only backend/EC2 should connect

---


