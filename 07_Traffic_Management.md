# 5. Traffic & Content Delivery Administration

Once you have your application running on multiple EC2 instances, you need a way to distribute user traffic to them efficiently and reliably. Furthermore, to ensure a fast experience for users worldwide, you need to deliver content from a location close to them. This chapter covers the administration of two key services that accomplish this: Elastic Load Balancing (ELB) and Amazon CloudFront.

---

## 5.1 Elastic Load Balancing (ELB)

[Elastic Load Balancing](https://docs.aws.amazon.com/elasticloadbalancing/latest/userguide/what-is-load-balancing.html) automatically distributes incoming application traffic across multiple targets, such as EC2 instances. It acts as the single "front door" for your application. We will use the **[Application Load Balancer (ALB)](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/introduction.html)**, which is best for HTTP and HTTPS traffic and provides advanced request routing.

### 5.1.1 Listeners and Target Groups

The ALB operates using a simple system of listeners and target groups.

*   **Listener:** A [listener](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/listeners.html) checks for connection requests from clients, using the protocol and port that you configure (e.g., HTTP on port 80). Each listener has a rule that forwards requests to a specific target group.
*   **Target Group:** A [target group](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-target-groups.html) is a collection of your EC2 instances. The load balancer routes requests to the registered targets in this group.

**Architectural Flow:**
`User Request -> Listener (e.g., Port 80) -> Rule -> Target Group -> Healthy EC2 Instance`

**Administrative Steps (AWS Console):**
1.  **Create a Target Group:**
    *   Navigate to the EC2 Dashboard and select "Target Groups" under "Load Balancing".
    *   Click "Create target group".
    *   Choose "Instances" as the target type.
    *   Give it a name (e.g., `app-targets`).
    *   Select your VPC. The protocol should be `HTTP` on port `80`.
    *   Proceed to the next step and register the EC2 instances you created in the previous chapter.
    *   Click "Create target group".

2.  **Create an Application Load Balancer:**
    *   Navigate to "Load Balancers" and click "Create Load Balancer".
    *   Choose "Application Load Balancer".
    *   Give it a name (e.g., `my-app-alb`).
    *   Set the scheme to **"Internet-facing"**.
    *   Under "Network mapping", select your VPC and map the ALB to your **two public subnets**. This is crucial for high availability.
    *   For "Security groups", select the `alb-sg` security group we designed earlier, which allows HTTP/HTTPS traffic from the internet.
    *   For "Listeners and routing", create a listener for `HTTP` on port `80`. Set the default action to forward to the `app-targets` target group you just created.
    *   Click "Create load balancer".

### 5.1.2 Health Checks

The load balancer periodically sends requests to its registered instances to test their status. If an instance fails these [health checks](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/target-group-health-checks.html), the load balancer automatically stops sending traffic to it and reroutes traffic to the remaining healthy instances.

**Administrative Steps:**
1.  When you create your target group, there is a "Health checks" section.
2.  **Health Check Protocol:** Use `HTTP`.
3.  **Health Check Path:** This should be a page on your application that returns a `200 OK` status code when the application is healthy. A common choice is `/` or `/healthcheck`. If this path returns an error or times out, the instance will be marked as unhealthy.
4.  **Advanced Settings:** You can configure thresholds, such as how many consecutive failed checks are required to mark an instance as unhealthy. The default settings are a good starting point.

---

## 5.2 Amazon CloudFront

[CloudFront](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/Introduction.html) is a Content Delivery Network (CDN) service. It speeds up the distribution of your static and dynamic web content to users by caching it in a global network of data centers called "edge locations". When a user requests content, they are routed to the edge location that provides the lowest latency, so content is delivered with the best possible performance.

### 5.2.1 Distributions and Origins

*   **Distribution:** A CloudFront [distribution](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/distribution-overview.html) tells CloudFront where you want to deliver content from.
*   **Origin:** An [origin](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/distribution-web-values-specify.html#DownloadDistValuesDomainName) is the source of your files. This can be an S3 bucket, an Elastic Load Balancer, or another HTTP server. We will configure two origins for our architecture.

**Administrative Steps (AWS Console):**
1.  Navigate to the CloudFront service dashboard.
2.  Click "Create a CloudFront distribution".

3.  **Configure the S3 Origin (for static assets):**
    *   In the "Origin domain" dropdown, select the S3 bucket you created for your application assets.
    *   CloudFront will recommend using "Origin access control settings". This is a feature that automatically creates and applies a bucket policy to your S3 bucket, ensuring that it can only be accessed by CloudFront. This is the recommended and most secure method.

4.  **Configure the ALB Origin (for dynamic content):**
    *   After creating the distribution, go to the "Origins" tab and click "Create origin".
    *   In the "Origin domain" dropdown, select the DNS name of the Application Load Balancer you created.

### 5.2.2 Cache Policies and Behaviors

[Behaviors](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/distribution-web-values-specify.html#DownloadDistValuesPathPattern) determine how CloudFront handles different types of requests. You can create rules based on the URL path pattern.

**Administrative Steps:**
1.  In your distribution settings, go to the "Behaviors" tab.
2.  **Default Behavior:** By default, a behavior is created for your primary origin (the S3 bucket). This is the "catch-all" behavior.
3.  **Create a New Behavior for Dynamic Content:**
    *   Click "Create behavior".
    *   **Path Pattern:** Enter a pattern that matches your application's dynamic routes (e.g., `/api/*`, `/login`).
    *   **Origin:** Select your Application Load Balancer origin.
    *   **Cache Policy:** For dynamic content that should not be cached, select a policy like `CachingDisabled`. For static assets, the default caching policy is usually a good starting point.
    *   This rule ensures that requests for `/api/*` are sent to your load balancer, while all other requests (for images, CSS, etc.) are sent to the S3 bucket.

By using both ELB and CloudFront, you create an architecture that is not only resilient and scalable but also fast and performant for users anywhere in the world.
