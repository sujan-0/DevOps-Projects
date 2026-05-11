---
title: "Deploying a Multi-Av.Z Private VPC Infrastructure on AWS "
datePublished: 2026-05-11T05:31:56.007Z
cuid: cmp0rldyb001g2ehy6f3ueaau
slug: deploying-a-multi-av-z-private-vpc-infrastructure-on-aws

---

**Published on:** 5/11/2026

I successfully completed my first major hands-on AWS project — building a **secure, scalable, and production-grade Multi-AZ VPC architecture**.

### **Project Overview**

I designed and deployed a complete **highly available** web application infrastructure following AWS best practices:

*   Application servers running in **private subnets** (no public IPs)
    
*   Secure access via **Bastion Host**
    
*   Automatic scaling and self-healing using **Auto Scaling Group**
    
*   Traffic distribution through **Application Load Balancer**
    
*   Multi-AZ deployment for high availability
    

### **Architecture Diagram**

![](https://cdn.hashnode.com/uploads/covers/67cf7a81a70907d5d60b29c7/10044005-932b-4173-8c42-8c173e39e7d3.png align="center")

### **Key AWS Services Used**

*   VPC + Public/Private Subnets
    
*   Internet Gateway & NAT Gateways
    
*   EC2 (Bastion + Application Servers)
    
*   Launch Template + Auto Scaling Group
    
*   Application Load Balancer (ALB)
    
*   Security Groups
    

### **Step-by-Step Implementation**

#### **1\. VPC Infrastructure Setup**

I used the **"VPC and more"** wizard to create:

*   VPC Name: `AWS-Prod-VPC`
    
*   2 Availability Zones
    
*   2 Public Subnets + 2 Private Subnets
    
*   2 NAT Gateways (one per AZ)
    

**Screenshots:**

![](https://cdn.hashnode.com/uploads/covers/67cf7a81a70907d5d60b29c7/a0e4eb71-830b-4cc0-a426-d3c629bbca68.png align="center")

#### **2\. Launch Template & Auto Scaling Group**

Created a Launch Template with:

*   AMI: Ubuntu Server 22.04 LTS
    
*   Instance Type: t2.micro
    
*   Security Group: `App-Server-SG`
    

**Security Group Rules:**

*   Port 22 (SSH) → Allowed only from Bastion Security Group
    
*   Port 8000 → Allowed from ALB Security Group
    

Then created **Auto Scaling Group** (`Prod-App-ASG`) with desired capacity = 2 in private subnets.

**Screenshots:**

*   Launch Template configuration  
    
    ![](https://cdn.hashnode.com/uploads/covers/67cf7a81a70907d5d60b29c7/d158865b-972c-496e-9a9c-c984c8879ba2.png align="center")
    
*   Security Group rules  
    
    ![](https://cdn.hashnode.com/uploads/covers/67cf7a81a70907d5d60b29c7/1e3de4d8-a748-42ac-b783-6849915f3a9a.png align="center")
    
*   Auto Scaling Group details
    
    ![](https://cdn.hashnode.com/uploads/covers/67cf7a81a70907d5d60b29c7/10626cd6-1208-4518-877e-e75d5239df5f.png align="center")
    
*   Private EC2 instances
    

![](https://cdn.hashnode.com/uploads/covers/67cf7a81a70907d5d60b29c7/ab1e8546-f901-40ad-8a30-64fd9e631197.png align="center")

#### **3\. Bastion Host Setup**

Launched one EC2 instance in a public subnet with public IP enabled. Configured its Security Group to allow SSH only from my IP address.

**Screenshots:**

*   Bastion EC2 details
    
    ![](https://cdn.hashnode.com/uploads/covers/67cf7a81a70907d5d60b29c7/85a1b138-f78b-44a4-a5aa-7a813d2a2f7a.png align="center")
    
*   Successful SSH: Local → Bastion → Private Instance  
    ssh -i your-key.pem image@ip-address
    

#### **4\. Deploying the Web Application**

On both private instances (via Bastion):

```bash
vim index.html
python3 -m http.server 8000 
```

**Screenshots:**

*   Commands executed
    
*   Running web server
    
*   Different pages from both instances
    
    ![](https://cdn.hashnode.com/uploads/covers/67cf7a81a70907d5d60b29c7/eb21900d-0fdc-4aaf-aa0a-f4a003fe3a06.png align="center")
    
    ![](https://cdn.hashnode.com/uploads/covers/67cf7a81a70907d5d60b29c7/c819314a-a73b-4229-a513-6aceef9589f3.png align="center")
    

#### **5\. Application Load Balancer Configuration**

*   Created Target Group (Instance type, Port 8000, Health check path `/`)
    
*   Launched Internet-facing ALB in public subnets
    
*   Listener: HTTP Port 80 forwarding to Target Group
    

**Screenshots:**

*   Target Group with Healthy targets  
    
    ![](https://cdn.hashnode.com/uploads/covers/67cf7a81a70907d5d60b29c7/b5e71257-c287-4c3c-b335-f76dec7221c9.png align="center")
    
*   ALB configuration & DNS name  
    
    ![](https://cdn.hashnode.com/uploads/covers/67cf7a81a70907d5d60b29c7/a3da011c-fe30-4b85-bd43-627d3f3ef9fe.png align="center")
    
*   **6\. Load Balancing Verification**
    

Accessed the ALB DNS name in the browser and refreshed multiple times. The page successfully alternated between Instance A and Instance B.

**Screenshots:**

*   Browser showing Instance A  
    
    ![](https://cdn.hashnode.com/uploads/covers/67cf7a81a70907d5d60b29c7/fc2d6eaa-dbfd-4c20-8a00-2c2150a96aa9.png align="center")
    
*   Browser showing Instance B  
    
    ![](https://cdn.hashnode.com/uploads/covers/67cf7a81a70907d5d60b29c7/0b31f16e-b24b-4028-99e4-14ac4e91d147.png align="center")
    

### **Challenges I Faced**

*   Key pair mismatch between Bastion and Launch Template instances
    
*   Not able to access bastion host due to keeping it on private-subnet (remember to select public).
    
*   Target Group showing unhealthy targets (solved by starting the Python server on port 8000)
    
*   Understanding proper Security Group configuration for private instances
    

### **Key Learnings**

*   Importance of **private subnets** and network isolation
    
*   How **Auto Scaling** and **Load Balancers** work together
    
*   Real-world security best practices (least privilege)
    
*   Difference between Public vs Private Subnets and routing
    

### **Conclusion**

This project gave me strong confidence in AWS networking and infrastructure. I now feel much more prepared to work on real-world cloud environments.

I’ll be documenting and sharing more cloud and DevOps projects in the coming weeks. Stay tuned!

**What do you think?** Feel free to share your thoughts or suggestions in the comments.

**#AWS #CloudComputing #VPC #DevOps #AutoScaling #LoadBalancer #CareerGrowth #LearningInPublic**