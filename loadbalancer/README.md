Question: Does Load balancer is placed in public subnet or private subnet?

It depends, If our load balancer is internet facing must be placed in the public subnets because it needs to connect directly to the internet to receive requests from your users

If application load balancer job is to route traffic inside your application layers (from Frontend to Backend). then it is placed pivate subnets because its only. 

<img width="720" height="590" alt="1_Z-DC516_NPhaBhs3o0VXVg" src="https://github.com/user-attachments/assets/68beede3-9d5f-4ab8-8ed7-b811d71829e0" />

https://medium.com/@aaloktrivedi/leveraging-high-availability-by-creating-an-aws-auto-scaling-group-application-load-balancer-2ea1f31a746










Good choice — **Load Balancer is VERY important** (used in almost every real system).
I’ll give you **clean, practical, DevOps-focused content + real labs** (not theory).

---

# 📘 `load-balancer/README.md`

# AWS Load Balancer (ALB / NLB)

A Load Balancer distributes incoming traffic across multiple servers to ensure:

* high availability
* scalability
* fault tolerance

👉 Instead of users hitting one server → traffic is distributed automatically.

---

## 🔹 What actually matters

Without Load Balancer:

```
User → EC2 (single point of failure ❌)
```

With Load Balancer:

```
User → Load Balancer → Multiple EC2 (high availability ✅)
```

👉 If one server fails → traffic goes to others.

---

## 🔹 Types of Load Balancer (simple)

### 1. Application Load Balancer (ALB)

* Works at **Layer 7 (HTTP/HTTPS)**
* Used for:

  * web apps
  * microservices
  * path-based routing

👉 Most commonly used in DevOps

---

### 2. Network Load Balancer (NLB)

* Works at **Layer 4 (TCP/UDP)**
* Very fast, low latency

👉 Used for high-performance systems

---

### 3. Gateway Load Balancer

* Used for security appliances (advanced)

---

## 🔹 Key Concepts

* **Listener** → listens on port (80 / 443)
* **Target Group** → group of servers (EC2, Pods)
* **Health Check** → checks if server is alive
* **Routing Rules** → decide where traffic goes

---

## 🔹 Real-world Architecture

```id="u6c9kf"
User → ALB → Target Group → EC2 / Containers
```

---

## 🔹 DevOps Use Case

In real pipelines:

* Jenkins builds app
* Docker image pushed
* App deployed on EC2 / Kubernetes
* Load Balancer exposes app

👉 Users NEVER access EC2 directly

---

## 🔹 Security Best Practices

* Use HTTPS (port 443)
* Attach SSL certificate (ACM)
* Restrict backend ports (only LB can access)
* Enable access logs
* Use WAF (Web Application Firewall)

---

## 🔹 Common Use Cases

* Hosting web applications
* Microservices routing
* Blue-Green deployment
* Canary releases
* Kubernetes Ingress

---



Commit Date: 26-April-2026
