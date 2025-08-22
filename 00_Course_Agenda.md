# Course Agenda: AWS Administration for High Availability

## Course Title: AWS Core Infrastructure for Administrators
**Target Audience:** Technical users familiar with IT concepts but new to AWS administration.
**Goal:** Configure the foundational AWS services required to deploy and manage a highly available application.

---

## Agenda: 1-Hour Technical Overview

### [1. Introduction & Architectural Goals](./01_Introduction_and_Architecture.md) (5 minutes)
*   **Objective:** Outline the target architecture for a highly available system.
*   **Key Concepts:** Fault tolerance, redundancy, scalability, and security.
*   **Diagram:** Present a clear architectural diagram showing the end-state: a multi-instance application behind a load balancer, with assets served by a CDN, all within a secure, isolated network.

### [2. Secure Network Foundation: VPC Configuration](./02_VPC_Configuration.md) (15 minutes)
*   **Title:** "VPC Administration: Isolating Your Resources"
*   **Objective:** To create a logically isolated network environment and control inbound/outbound traffic.
*   **Administrative Tasks:**
    *   **CIDR Block Planning:** Choosing an appropriate IP address range.
    *   **Subnetting:** Creating public subnets for web-facing resources and private subnets for backend services and databases.
    *   **Routing:** Configuring Route Tables, an Internet Gateway for public traffic, and a NAT Gateway for private instances to access the internet.
    *   **Security Layers:** Differentiating between stateless Network ACLs (subnet level) and stateful Security Groups (instance level).

### [3. Virtual Servers: EC2 Instance Management](./03_EC2_Instance_Management.md) (15 minutes)
*   **Title:** "EC2 Administration: Deploying & Securing Compute"
*   **Objective:** To deploy, secure, and manage the virtual servers that run the application.
*   **Administrative Tasks:**
    *   **Instance Selection:** Choosing the right instance type and family based on workload (compute-optimized, memory-optimized, etc.).
    *   **AMI Management:** Using Amazon Machine Images (AMIs) for creating consistent, repeatable server templates.
    *   **Access Control:** Implementing Security Groups to act as instance-level firewalls.
    *   **Key Pair Management:** Creating and managing SSH keys for secure administrative access.
    *   **Bootstrapping:** Using "User Data" scripts to automate initial server configuration on launch.

### [4. IAM for Resource Access: Granting Permissions with Roles](./04_IAM_for_Resource_Access.md) (10 minutes)
*   **Title:** "IAM for Resource Access: Granting Permissions with Roles"
*   **Objective:** To understand and implement the secure way for AWS resources to interact with each other.
*   **Administrative Tasks:**
    *   Creating IAM Policies with specific permissions.
    *   Creating IAM Roles for the EC2 service.
    *   Attaching Roles to EC2 instances via an Instance Profile.

### [5. Scalable Object Storage: S3 Bucket Administration](./05_S3_Bucket_Administration.md) (5 minutes)
*   **Title:** "S3 Administration: Managing Application Assets"
*   **Objective:** To configure durable, scalable, and secure storage for application assets, logs, and backups.
*   **Administrative Tasks:**
    *   **Bucket Configuration:** Creating and naming S3 buckets.
    *   **Access Control:** Managing Block Public Access settings and introduction to bucket policies.
    *   **Storage Classes:** Understanding the use cases for different storage tiers (e.g., S3 Standard vs. Glacier).

### [6. Traffic Management: Elastic Load Balancing & CloudFront](./06_Traffic_Management.md) (10 minutes)
*   **Title:** "Traffic & Content Delivery Administration"
*   **Objective:** To distribute traffic for scalability and reliability, and to accelerate content delivery.
*   **Administrative Tasks (ELB):**
    *   **Listeners & Target Groups:** Configuring the load balancer to listen for traffic and forward it to registered EC2 instances.
    *   **Health Checks:** Implementing health checks to automatically remove unhealthy instances from the pool.
*   **Administrative Tasks (CloudFront):**
    *   **Distributions:** Creating a CDN distribution.
    *   **Origins:** Configuring origins, such as an S3 bucket for static assets or a load balancer for dynamic content.
    *   **Cache Policies:** Understanding cache behavior and Time-To-Live (TTL) settings.

### [7. Summary & Next Steps](./07_Summary_and_Next_Steps.md) (10 minutes)
*   **Title:** "Operationalizing Your AWS Environment"
*   **Review:** Briefly recap how the services connect to achieve high availability.
*   **Next Steps:** Introduce subsequent topics for deeper dives:
    *   Identity and Access Management (IAM) for granular permissions.
    *   Automating deployments with Infrastructure as Code (e.g., CloudFormation or Terraform).
    *   Monitoring and logging with CloudWatch.
*   **Q&A.**

### [8. Recommended AWS Educate Courses](./08_AWS_Educate_Courses.md)
*   A list of supplementary, hands-on courses from AWS Educate to reinforce the concepts in this guide.
