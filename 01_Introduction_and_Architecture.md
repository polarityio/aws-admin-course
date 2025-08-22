# 1. Introduction & Architectural Goals

## Objective
The primary objective of this guide is to provide the administrative knowledge required to deploy a highly available, scalable, and secure application in the AWS cloud. By the end of this course, you will be able to configure the core AWS services to build a resilient infrastructure, ensuring your application has minimal downtime and can adapt to changing user demand.

---

## Core Concepts for High Availability

To build a robust AWS environment, it is essential to understand the following foundational principles:

*   **Fault Tolerance:** This is the ability of a system to remain in operation even if some of its components fail. In AWS, this is achieved by designing systems with no single point of failure. For example, if one virtual server (EC2 instance) fails, another identical one should be available to take over its workload seamlessly, without any impact on the end-user.

*   **Redundancy:** Redundancy is the practice of duplicating critical components of a system to increase reliability. We achieve fault tolerance *through* redundancy. In AWS, this often means running multiple EC2 instances in different physical locations (known as Availability Zones) or replicating data across multiple storage devices in Amazon S3.

*   **Scalability:** This refers to the ability of a system to handle a growing amount of work by adding resources.
    *   **Vertical Scaling:** Increasing the size and power of a single resource (e.g., upgrading an EC2 instance to one with more CPU and RAM). This is simple but has limits.
    *   **Horizontal Scaling (Elasticity):** Increasing the number of resources (e.g., adding more EC2 instances). This is the preferred method in the cloud as it is more flexible and fault-tolerant. AWS services like Auto Scaling allow this to happen automatically based on traffic.

*   **Security:** Security in the cloud is a shared responsibility. AWS is responsible for the security *of* the cloud (protecting the physical infrastructure), while you, the customer, are responsible for security *in* the cloud. This includes:
    *   **Network Security:** Configuring firewalls (Security Groups, NACLs) to control traffic to your resources.
    *   **Access Control:** Ensuring only authorized personnel have access to your AWS resources using services like IAM.
    *   **Data Protection:** Encrypting data both in transit (as it moves over a network) and at rest (as it is stored).

---

## Target Architecture Diagram

The infrastructure we will build is designed to achieve these core concepts. Below is a description of the target architecture.

*(Note: A visual diagram should be inserted here to illustrate the connections between the services.)*

### Architectural Description:

1.  **Region & VPC:** The entire environment is contained within a single [AWS Region](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-regions-availability-zones.html) and is logically isolated from other networks by a **[Virtual Private Cloud (VPC)](https://docs.aws.amazon.com/vpc/latest/userguide/what-is-amazon-vpc.html)**.

2.  **Availability Zones:** The VPC spans at least two [Availability Zones (AZs)](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-regions-availability-zones.html#concepts-availability-zones). Each AZ is a distinct physical data center, providing redundancy. If one AZ has an outage, our application will continue to run in the other.

3.  **Network:**
    *   The VPC has both **public and private subnets** in each AZ.
    *   An **[Internet Gateway](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Internet_Gateway.html)** is attached to the VPC to allow traffic from the internet to the public subnets.
    *   **Public Subnets** contain resources that need to be directly accessible from the internet, such as our Load Balancer.
    *   **Private Subnets** contain our backend application servers (EC2 instances), which should not be directly exposed to the internet.
    *   A **[NAT Gateway](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-gateway.html)** resides in the public subnet, allowing instances in the private subnets to initiate outbound traffic to the internet (e.g., for software updates) without allowing inbound connections.

4.  **Application Tier:**
    *   An **[Application Load Balancer (ALB)](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/introduction.html)** serves as the single entry point for all user traffic. It is deployed across the public subnets in both AZs.
    *   The ALB distributes incoming traffic to a fleet of **[EC2 instances](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/concepts.html)** running the application. These instances are in an **[Auto Scaling Group](https://docs.aws.amazon.com/autoscaling/ec2/userguide/auto-scaling-groups.html)**.
    *   The EC2 instances are located in the **private subnets**. They only accept traffic from the Load Balancer, not directly from the internet.
    *   The **Auto Scaling Group** automatically adds or removes EC2 instances based on demand (e.g., CPU utilization), ensuring performance and cost-efficiency.

5.  **Storage & Content Delivery:**
    *   Static assets like images, videos, and CSS files are stored in an **[Amazon S3 bucket](https://docs.aws.amazon.com/AmazonS3/latest/userguide/UsingBuckets.html)**.
    *   An **[Amazon CloudFront distribution](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/Introduction.html)** is set up to cache the content from the S3 bucket at edge locations around the world. This provides low-latency content delivery to users globally and reduces the load on our application servers.

This architecture creates a system that is resilient to component failure, can scale automatically, and is built on a secure network foundation.
