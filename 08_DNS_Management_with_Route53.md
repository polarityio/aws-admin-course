# 8. DNS Management with Route 53

So far, your application is accessible via the long, auto-generated DNS name of your CloudFront distribution or Application Load Balancer. To provide a professional and user-friendly experience, you need to point your custom domain name (e.g., `www.your-company.com`) to your AWS resources. The service that makes this possible is **Amazon Route 53**.

---

## 8.1 What is Amazon Route 53?

[Amazon Route 53](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/Welcome.html) is a highly available and scalable cloud Domain Name System (DNS) web service. It is designed to give developers and businesses an extremely reliable and cost-effective way to route end users to Internet applications.

**Key Functions:**

*   **Domain Registration:** You can register new domain names directly through Route 53.
*   **DNS Routing:** It translates human-readable domain names into the numeric IP addresses that computers use to connect to each other. This is its primary function for our purposes.
*   **Health Checking:** Route 53 can monitor the health of your application and automatically route traffic away from unhealthy endpoints to healthy ones.

---

## 8.2 Administrative Steps: Pointing a Domain to Your Application

The process involves two main steps: creating a "Hosted Zone" for your domain in Route 53, and then creating "Record Sets" within that zone to direct traffic.

### Step 1: Create a Hosted Zone
A hosted zone is a container that holds the DNS records for a specific domain.

1.  Navigate to the **Route 53** service in the AWS Console.
2.  If you don't own the domain name yet, you can register one under "Registered domains". If you already own a domain (e.g., you bought it from another registrar like GoDaddy or Namecheap), proceed to the next step.
3.  In the dashboard, click on **Hosted zones**.
4.  Click **Create hosted zone**.
5.  **Domain name:** Enter your domain name (e.g., `your-company.com`).
6.  **Type:** Select **Public hosted zone**.
7.  Click **Create hosted zone**.

After creating the hosted zone, Route 53 will assign four "Name Server" (NS) records to it. You must update your domain registrar's settings to use these four AWS name servers. This step delegates the authority for your domain's DNS to Amazon Route 53. The process for this varies by registrar, but is a standard and well-documented procedure.

### Step 2: Create a DNS Record
Once your hosted zone is set up, you can create records to point your domain to your AWS resources. The best practice is to use an **Alias record**. An Alias record is a special Route 53 feature that lets you route traffic to selected AWS resources, like a CloudFront distribution or a Load Balancer.

**Creating an Alias Record:**
1.  Inside your newly created hosted zone, click **Create record**.
2.  **Record name:**
    *   To create a record for the root domain (`your-company.com`), leave this blank.
    *   To create a record for a subdomain (like `www.your-company.com`), enter `www`.
3.  **Record type:** Select **A - Routes traffic to an IPv4 address and some AWS resources**.
4.  **Route traffic to:**
    *   Enable the **Alias** toggle.
    *   In the dropdown menu, choose the appropriate endpoint. For our architecture, this would be **"Alias to CloudFront distribution"**.
    *   Another dropdown will appear, and you can select your specific CloudFront distribution from the list. (You can similarly point it to your Application Load Balancer).
5.  Click **Create records**.

After a short propagation time, anyone who navigates to your custom domain in a web browser will be directed to your CloudFront distribution, which will then serve content from your S3 bucket and Application Load Balancer. You have now successfully connected your user-friendly domain name to your highly available AWS infrastructure.
