# K-Store: DevOps E-Commerce Platform on AWS

**Date:** July 2025  
**Tech Stack:** AWS (ECS, EC2, RDS, EFS, VPC), Docker, MariaDB, PrestaShop  
**Status:** Completed

## 📖 Project Overview

**K-Store** is a resilient, auto-scaling e-commerce platform architected on AWS. This project demonstrates the migration of a monolithic PrestaShop application into a containerized, cloud-native environment using **Amazon ECS**.

The solution was designed to meet strict requirements for High Availability (HA), security (Defense in Depth), and scalability to support a simulated expansion into the Asia Pacific region.

### Key Features
* **Container Orchestration:** Deployed PrestaShop using AWS ECS with an EC2 launch type.
* **High Availability:** Achieved redundancy across multiple Availability Zones (AZs) with an Application Load Balancer (ALB).
* **Security:** Implemented a custom VPC with public/private subnet isolation, strict Security Groups, and encrypted data at rest and in transit.
* **State Management:** Solved container state issues using Amazon EFS for static content and Amazon RDS (MariaDB) for transactional data.

---

## 🏢 Business Scenario & Requirements

**Client:** Kaperski Pte Ltd (Startup)  
**Objective:** Rapidly deploy a secure, cost-effective online shopping infrastructure capable of scaling for APAC expansion.

**Key Requirements:**
1.  **High Availability:** Service must be available 24x7.
2.  **Scalability:** Infrastructure must scale efficiently to accommodate customer growth.
3.  **Security:** Data encryption (at rest/transit), secured access controls, and network segmentation.
4.  **Cost Control:** Leverage AWS Managed Services to reduce operational overhead.

---

## 🏗️ Architecture Design

The architecture follows AWS best practices for a 3-tier web application, leveraging managed services to reduce operational overhead.

![Architecture Diagram]<img width="1443" height="890" alt="C3349C drawio white" src="https://github.com/user-attachments/assets/16b3eda6-527f-445b-a20b-829767390ce3" />


### AWS Services Implemented

| Service | Component | Usage Description |
| :--- | :--- | :--- |
| **Amazon VPC** | Networking | Custom VPC with Public (DMZ) and Private subnets for network isolation. |
| **Amazon ECS** | Orchestration | Manages Docker containers on `t3.small` EC2 instances via Auto Scaling Groups. |
| **Amazon RDS** | Database | Fully managed **MariaDB** instance with encryption at rest for transactional data. |
| **Amazon EFS** | Storage | Shared file system for persisting PrestaShop static content (images, themes) across dynamic containers. |
| **ALB** | Load Balancing | Application Load Balancer handling traffic distribution and SSL termination. |
| **Route 53 & ACM** | DNS & Security | Custom domain management (`kenny-twk-api-aws.click`) and public SSL/TLS certificates. |
| **Amazon ECR** | Registry | Private repository for custom-built Docker images with optimized PHP configurations. |

---

## 🛡️ Security Strategy

Security was applied using a **"Defense in Depth"** strategy, ensuring no single point of failure compromise.

![Security Diagram]<img width="593" height="891" alt="Picture1" src="https://github.com/user-attachments/assets/e0a3932d-ef8c-45ad-ba07-dcc4d06af47f" />

*(Note: Upload your security diagram to your repo and update this path)*

### 1. Network Isolation
* **Public Subnets:** Only house the Application Load Balancer (ALB).
* **Private Subnets:** House critical resources (EC2, RDS, EFS). No direct internet access; outbound updates flow through a NAT Gateway.

### 2. Traffic Control (Security Groups)
Traffic is strictly controlled via "Allow" rules (Deny by default):
* **ALB:** Accepts HTTPS (443) from `0.0.0.0/0`.
* **App Servers (ECS):** Only accept HTTP (80) from the *ALB Security Group*.
* **Database (RDS):** Only accepts MySQL (3306) from the *App Server Security Group*.
* **Storage (EFS):** Only accepts NFS (2049) from the *App Server Security Group*.

### 3. Identity & Access (IAM)
* **Principle of Least Privilege:** Custom IAM roles (`ecsTaskExecutionRole`) were created to allow ECS tasks to pull images from ECR and push logs to CloudWatch without over-provisioning permissions.
* **Secure Access:** SSH ports (22) are closed. Access to instances is managed via **AWS Systems Manager Session Manager**.

---

## 🚀 Deployment Workflow

The project followed a phased deployment strategy to ensure stability:

1.  **Local Prototype:**
    * Containerization of PrestaShop (v9.0.0) using Docker.
    * Verified database connectivity and static file handling locally.
2.  **Cloud Infrastructure Setup:**
    * Provisioning of VPC, subnets, and route tables.
    * Setup of RDS and EFS to handle state.
3.  **Container Migration:**
    * Custom Docker image creation (optimizing PHP memory limits).
    * Pushing image to Amazon ECR.
4.  **Orchestration:**
    * Configuring ECS Task Definitions and Services.
    * Connecting the Service to the ALB and Auto Scaling Group.

---

## ⚠️ Challenges & Future Roadmap

This project successfully implemented core cloud principles. However, the following limitations were identified during the review, with proposed solutions for the next iteration.

| Limitation | Impact | Proposed Solution |
| :--- | :--- | :--- |
| **Manual Deployment** | High risk of human error; difficult to replicate across environments. | **Infrastructure as Code (IaC):** Migrate deployment to Terraform or AWS CloudFormation. |
| **Stateful Installation** | Long boot times and race conditions during auto-scaling events. | **Golden Image Strategy:** "Bake" the installed application state into the Docker image to make containers fully stateless. |
| **Secrets Management** | DB passwords stored in Task Definitions (Plain Text). | **AWS Secrets Manager:** Inject credentials at runtime securely via IAM authentication. |
| **EFS Latency** | Network storage can be slower than local disk for high-I/O PHP apps. | **Caching Strategy:** Implement Redis/ElastiCache for session and query caching. |

---
