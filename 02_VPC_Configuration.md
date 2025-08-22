# 2. VPC Administration: Isolating Your Resources

A [Virtual Private Cloud (VPC)](https://docs.aws.amazon.com/vpc/latest/userguide/what-is-amazon-vpc.html) is the network foundation of your AWS environment. It is a logically isolated section of the AWS Cloud where you can launch AWS resources in a virtual network that you define. You have complete control over your virtual networking environment, including selection of your own IP address range, creation of subnets, and configuration of route tables and network gateways.

This chapter covers the administrative tasks required to build the VPC for our target architecture.

---

## 2.1 CIDR Block Planning

Before creating a VPC, you must specify a range of IPv4 addresses for it in the form of a [Classless Inter-Domain Routing (CIDR) block](https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing). This is the primary, private IP address range for your entire VPC.

**Key Considerations:**

*   **Size:** The CIDR block size can be between `/16` (65,536 IP addresses) and `/28` (16 IP addresses). It's best practice to provision a larger VPC (like `/16`) to ensure you don't run out of IP addresses as your application grows. You cannot change the primary CIDR block of a VPC after it has been created.
*   **Non-Overlapping:** The CIDR block you choose must **not** overlap with any other network you might connect to. This includes your on-premises corporate network or other VPCs you might peer with in the future. A common and safe choice for private networks is to use a range from [RFC 1918](https://tools.ietf.org/html/rfc1918) (e.g., `10.0.0.0/16`, `172.16.0.0/16`, `192.168.0.0/16`).

**For our architecture, we will use `10.0.0.0/16`.** This gives us ample room for future expansion.

---

## 2.2 Subnetting

A [subnet](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Subnets.html) is a range of IP addresses within your VPC. You will launch your AWS resources, like EC2 instances, into these subnets. We will create public and private subnets to separate our resources based on whether they need to be accessible from the internet.

*   **Public Subnets:** Resources in a public subnet can have direct access to the internet via the Internet Gateway. Our load balancers will reside here.
*   **Private Subnets:** Resources in a private subnet are not directly accessible from the internet. Our application servers will reside here for security.

For high availability, we will create one public and one private subnet in at least two [Availability Zones (AZs)](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-regions-availability-zones.html#concepts-availability-zones).

**Subnet CIDR Block Allocation:**

We will divide our `10.0.0.0/16` VPC block into smaller `/24` blocks for our subnets. A `/24` block contains 256 IP addresses.

*   **AZ 1:**
    *   Public Subnet 1: `10.0.1.0/24`
    *   Private Subnet 1: `10.0.2.0/24`
*   **AZ 2:**
    *   Public Subnet 2: `10.0.3.0/24`
    *   Private Subnet 2: `10.0.4.0/24`

**Administrative Steps (AWS Console):**
1.  Navigate to the VPC Dashboard.
2.  Select "Your VPCs" and click "Create VPC".
3.  Specify the name and our chosen CIDR block (`10.0.0.0/16`).
4.  Once the VPC is created, navigate to the "Subnets" section.
5.  Click "Create subnet" for each of the four subnets listed above, ensuring you select the correct VPC, assign them to the appropriate Availability Zone, and specify their unique CIDR block.

---

## 2.3 Routing

Routing determines where network traffic from your subnets is directed. Each subnet must be associated with a [route table](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Route_Tables.html).

### Internet Gateway (IGW)
An [Internet Gateway](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Internet_Gateway.html) is a horizontally scaled, redundant, and highly available VPC component that allows communication between your VPC and the internet.

**Administrative Steps:**
1.  In the VPC Dashboard, go to "Internet Gateways" and click "Create internet gateway".
2.  Give it a name and create it.
3.  Select the newly created IGW and, from the "Actions" menu, choose "Attach to VPC". Select your VPC.

### Public Route Table
We need a route table that directs internet-bound traffic to the IGW.

**Administrative Steps:**
1.  Go to "Route Tables" and "Create route table". Name it (e.g., `Public-RT`) and select your VPC.
2.  Select the new route table and go to the "Routes" tab. Click "Edit routes".
3.  Add a new route:
    *   **Destination:** `0.0.0.0/0` (This means "all traffic not destined for another location within the VPC").
    *   **Target:** Select "Internet Gateway" and choose the IGW you created.
4.  Go to the "Subnet Associations" tab. Click "Edit subnet associations".
5.  Associate this route table with your **two public subnets** (`10.0.1.0/24` and `10.0.3.0/24`).

### NAT Gateway
A [Network Address Translation (NAT) Gateway](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-gateway.html) enables instances in a private subnet to connect to the internet or other AWS services, but prevents the internet from initiating a connection with those instances. This is crucial for tasks like downloading software updates.

**Administrative Steps:**
1.  A NAT Gateway needs an Elastic IP address. Go to "Elastic IPs" and "Allocate Elastic IP address".
2.  Go to "NAT Gateways" and "Create NAT Gateway".
3.  Give it a name, select one of your **public subnets** for it to reside in, and assign the Elastic IP you just created.
4.  Create a new, separate route table for your private subnets (e.g., `Private-RT`).
5.  In this new private route table, add a route:
    *   **Destination:** `0.0.0.0/0`
    *   **Target:** Select "NAT Gateway" and choose the one you created.
6.  Associate this private route table with your **two private subnets** (`10.0.2.0/24` and `10.0.4.0/24`).

---

## 2.4 Security Layers

AWS provides two layers of firewall protection: Network ACLs and Security Groups.

### Network ACLs (NACLs)
*   **Scope:** Operates at the **subnet** level.
*   **Rules:** Can have both "allow" and "deny" rules.
*   **State:** **Stateless**. This means return traffic must be explicitly allowed by a rule. If you allow inbound traffic on a certain port, you must also create a rule to allow outbound traffic on that port.
*   **Default:** The default NACL allows all inbound and outbound traffic. It's best practice to leave the default NACL as is and use Security Groups for primary protection unless you have a specific need for subnet-level blocking.
*   **Further Reading:** [Network ACLs Documentation](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-network-acls.html)

### Security Groups (SGs)
*   **Scope:** Operates at the **instance** (EC2) level.
*   **Rules:** Can only have "allow" rules. There are no "deny" rules.
*   **State:** **Stateful**. This means if you allow inbound traffic, the return traffic is automatically allowed, regardless of any outbound rules. This makes them much easier to manage.
*   **Default:** The default SG allows all outbound traffic but denies all inbound traffic until you add rules.
*   **Further Reading:** [Security Groups Documentation](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-security-groups.html)

For our architecture, we will rely primarily on Security Groups for their stateful nature and instance-level granularity. We will configure these in the next chapter when we set up our EC2 instances and Load Balancer.
