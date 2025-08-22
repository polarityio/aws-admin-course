# 3. EC2 Administration: Deploying & Securing Compute

[Amazon Elastic Compute Cloud (EC2)](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/concepts.html) provides scalable computing capacity in the AWS Cloud. It is the service you use to create and manage virtual servers, which are called "instances". This chapter covers the essential administrative tasks for deploying the EC2 instances that will run our application, focusing on consistency, security, and automation.

---

## 3.1 Instance Selection

Choosing the correct [instance type](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instance-types.html) is a critical step that impacts performance and cost. AWS offers a wide variety of instance types optimized for different use cases.

**Instance Families:**

*   **General Purpose (e.g., T-series, M-series):** Provide a balance of compute, memory, and networking resources. Excellent for a wide range of workloads like web servers and small-to-mid-sized databases. The T-series are "burstable," offering a low-cost baseline performance with the ability to burst to higher performance when needed.
*   **Compute Optimized (e.g., C-series):** Ideal for compute-intensive applications like high-performance web servers, scientific modeling, and batch processing.
*   **Memory Optimized (e.g., R-series, X-series):** Designed for workloads that process large data sets in memory, such as high-performance databases or real-time big data analytics.
*   **Storage Optimized (e.g., I-series, D-series):** Designed for workloads that require high, sequential read and write access to very large data sets on local storage, like data warehousing.

**Selection Strategy:**
For a typical web application, it is best to start with a **General Purpose** instance type (e.g., `t3.micro` or `t3.small`) and monitor its performance using CloudWatch. You can resize the instance later if needed. For our initial setup, we will use `t3.micro` as it is eligible for the [AWS Free Tier](https://aws.amazon.com/free/) and is sufficient for a basic application.

---

## 3.2 AMI Management

An [Amazon Machine Image (AMI)](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AMIs.html) is a template that contains the software configuration (operating system, application server, and applications) required to launch your instance.

**Importance of AMIs:**

*   **Consistency:** AMIs ensure that every instance you launch is an identical clone, which is critical for scalability and troubleshooting.
*   **Speed:** Pre-configuring your software in an AMI significantly reduces the boot time of new instances.
*   **Repeatability:** You can easily launch instances in different regions or for different environments (dev, test, prod) using the same AMI.

**Administrative Strategy:**
1.  **Start with a Base AMI:** Begin with a standard AWS-provided AMI (e.g., Amazon Linux 2, Ubuntu Server).
2.  **Create a "Golden AMI":**
    *   Launch a temporary instance from the base AMI.
    *   Install all necessary software, patches, and your application code on this instance.
    *   Stop the instance to ensure data integrity.
    *   From the EC2 console, select the instance and choose "Actions" -> "Image and templates" -> "Create image".
    *   This creates your custom "golden AMI".
3.  **Launch Instances from the Golden AMI:** Use this new AMI to launch all instances for your application. When you need to update your application or apply patches, you repeat the process to create a new version of your golden AMI.

---

## 3.3 Access Control: Security Groups

As discussed in the VPC chapter, [Security Groups](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-security-groups.html) act as a stateful firewall for your instances. We will create two critical Security Groups for our architecture.

**1. Web Server Security Group (`web-sg`)**
This SG will be attached to our EC2 instances. It should only allow traffic from the load balancer.

**Inbound Rules for `web-sg`:**
*   **Type:** `HTTP`
*   **Protocol:** `TCP`
*   **Port Range:** `80`
*   **Source:** The Security Group of our Application Load Balancer (which we will create in a later chapter, e.g., `alb-sg`). This is a key security practice: by specifying another SG as the source, you ensure that only the load balancer can send traffic to your instances, not the entire internet.
*   **Type:** `SSH`
*   **Protocol:** `TCP`
*   **Port Range:** `22`
*   **Source:** Your corporate IP address or a bastion host's IP. This should be locked down for administrative access and should **never** be open to the world (`0.0.0.0/0`).

**2. Load Balancer Security Group (`alb-sg`)**
This SG will be attached to our Application Load Balancer. It allows web traffic from the internet.

**Inbound Rules for `alb-sg`:**
*   **Type:** `HTTP`
*   **Protocol:** `TCP`
*   **Port Range:** `80`
*   **Source:** `0.0.0.0/0` (Any IPv4 address)
*   **Type:** `HTTPS`
*   **Protocol:** `TCP`
*   **Port Range:** `443`
*   **Source:** `0.0.0.0/0` (Any IPv4 address)

---

## 3.4 Key Pair Management

Amazon EC2 uses public-key cryptography to encrypt and decrypt login information. To log in to your instance, you must create a [key pair](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html), specify the name of the key pair when you launch the instance, and provide the private key when you connect.

**Administrative Steps:**
1.  In the EC2 Dashboard, navigate to "Network & Security" -> "Key Pairs".
2.  Click "Create key pair".
3.  Give it a name (e.g., `app-admin-key`).
4.  Choose the private key file format (`.pem` for OpenSSH or `.ppk` for PuTTY).
5.  Click "Create key pair". The private key will be downloaded automatically.
6.  **Crucially, store this private key in a secure location. You cannot download it again. If you lose it, you will lose administrative access to any instance launched with it.**

When you launch a new EC2 instance, you will be prompted to select a key pair. Choose the one you just created.

---

## 3.5 Bootstrapping with User Data

[User Data](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/user-data.html) is a script that is automatically executed when you launch an instance for the first time. This is a powerful tool for automating the final steps of configuration.

**Common Use Cases:**
*   Pulling the latest version of application code from a repository.
*   Starting services or daemons.
*   Applying configuration settings that are specific to an environment (e.g., dev vs. prod).

**Example User Data Script (for Amazon Linux 2):**
This script updates the server, installs a simple Apache web server, and starts the service. You would enter this in the "User Data" field in the "Advanced Details" section during instance launch.

```bash
#!/bin/bash
yum update -y
yum install -y httpd
systemctl start httpd
systemctl enable httpd
```

By combining a pre-configured Golden AMI with a User Data script, you can create a fully automated process for launching ready-to-serve application instances.
