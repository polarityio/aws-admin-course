# 4. S3 Administration: Managing Application Assets

[Amazon Simple Storage Service (S3)](https://docs.aws.amazon.com/AmazonS3/latest/userguide/Welcome.html) is an object storage service that offers industry-leading scalability, data availability, security, and performance. It is the ideal place to store application assets such as images, videos, documents, logs, and backups. Unlike the block storage used for EC2 instances (EBS), S3 is designed for extreme durability and accessibility from anywhere.

This chapter covers the fundamental administrative tasks for configuring and securing S3 buckets.

---

## 4.1 Bucket Configuration

An S3 ["bucket"](https://docs.aws.amazon.com/AmazonS3/latest/userguide/UsingBuckets.html) is a container for your data. You can think of it as a top-level folder.

**Key Configuration Rules:**

*   **Globally Unique Name:** A bucket name must be globally unique across all of AWS. No two accounts can have a bucket with the same name. It's a common practice to prefix bucket names with your company name or an abbreviation to ensure uniqueness (e.g., `mycompany-app-assets`).
*   **Naming Conventions:** Bucket names must be between 3 and 63 characters long and can consist only of lowercase letters, numbers, dots (.), and hyphens (-). [Read the full naming rules here](https://docs.aws.amazon.com/AmazonS3/latest/userguide/bucketnamingrules.html).
*   **Region Selection:** You choose the AWS Region where the bucket will be created. To minimize latency, choose the region closest to your users or your EC2 instances.

**Administrative Steps (AWS Console):**
1.  Navigate to the S3 service dashboard.
2.  Click "Create bucket".
3.  Enter a globally unique bucket name.
4.  Select the desired AWS Region.
5.  Leave the "Block Public Access" settings at their default (enabled). We will discuss this next.
6.  Optionally, enable ["Bucket Versioning"](https://docs.aws.amazon.com/AmazonS3/latest/userguide/Versioning.html). Versioning keeps a complete history of all changes to an object, which is an excellent safety measure against accidental deletions or overwrites.
7.  Click "Create bucket".

---

## 4.2 Access Control

By default, all S3 buckets and the objects within them are **private**. Only the root user of the AWS account that created the bucket can access it. Securing your data is the most critical aspect of S3 administration.

### Block Public Access
This is a set of security controls at the account and bucket level that helps you ensure your data is never exposed to the public internet unless you explicitly intend it to be. The default setting is to **block all public access**, and it is highly recommended to leave this enabled. [Learn more about Blocking Public Access here](https://docs.aws.amazon.com/AmazonS3/latest/userguide/access-control-block-public-access.html).

Our application's EC2 instances will access the bucket's content using IAM Roles (covered in detail in the [IAM for Resource Access](./04_IAM_for_Resource_Access.md) chapter), not via public access. Our users will access specific assets via CloudFront, which will be granted special permission to retrieve content from our private S3 bucket.

### Bucket Policies
[Bucket policies](https://docs.aws.amazon.com/AmazonS3/latest/userguide/bucket-policies.html) are JSON-based documents that give you fine-grained control over access to your S3 bucket. You can specify who can access the bucket, which actions they can perform (e.g., `s3:GetObject`, `s3:PutObject`), and from where (e.g., only from a specific IP address or VPC).

While a full discussion of bucket policies is extensive, a common use case is to grant read-only access to a CloudFront distribution, which we will do in the CloudFront chapter.

---

## 4.3 Storage Classes

S3 offers a range of [storage classes](https://docs.aws.amazon.com/AmazonS3/latest/userguide/storage-class-intro.html) designed for different use cases and cost profiles. Choosing the right one can significantly reduce your storage costs.

**Common Storage Classes:**

*   **S3 Standard:**
    *   **Use Case:** The default class for frequently accessed data. Ideal for dynamic websites, content distribution, and mobile/gaming applications.
    *   **Characteristics:** High durability, availability, and performance. It is the most expensive class.

*   **S3 Intelligent-Tiering:**
    *   **Use Case:** For data with unknown or changing access patterns.
    *   **Characteristics:** S3 automatically moves your data to the most cost-effective access tier based on how frequently you use it. There is a small monthly monitoring and automation fee per object.

*   **S3 Standard-Infrequent Access (S3 Standard-IA):**
    *   **Use Case:** For data that is accessed less frequently but requires rapid access when needed. Good for long-term storage, backups, and disaster recovery files.
    *   **Characteristics:** Same durability as S3 Standard, but with a lower per-GB storage price and a per-GB retrieval fee.

*   **Amazon S3 Glacier (Instant Retrieval, Flexible Retrieval, Deep Archive):**
    *   **Use Case:** Long-term archival.
    *   **Characteristics:** These are the lowest-cost storage classes. They are designed for data that is rarely accessed and can tolerate retrieval times ranging from milliseconds (Instant Retrieval) to several hours (Deep Archive).

**Administrative Strategy:**
For our application assets, which will be served to users via CloudFront, **S3 Standard** is the appropriate choice. For other data like application logs or backups, consider using S3 Standard-IA or S3 Intelligent-Tiering to optimize costs. You can set up ["Lifecycle Policies"](https://docs.aws.amazon.com/AmazonS3/latest/userguide/lifecycle-configuration-intro.html) in S3 to automatically transition objects between storage classes after a certain period.
