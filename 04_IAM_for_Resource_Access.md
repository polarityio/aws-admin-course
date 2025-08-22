# 4. IAM for Resource Access: Granting Permissions with Roles

A core principle of AWS security is to avoid using static, long-lived credentials (like access keys) whenever possible. When an application or service running on an EC2 instance needs to access other AWS resources (like an S3 bucket), the most secure and manageable way to grant those permissions is by using an **IAM Role**.

---

## 4.1 What is an IAM Role?

An [IAM Role](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles.html) is an IAM identity that you can create in your account that has specific permissions. It is similar to an IAM user in that it is an AWS identity with permission policies that determine what the identity can and cannot do in AWS. However, instead of being uniquely associated with one person, a role is intended to be assumable by anyone who needs it.

Most importantly, an IAM Role does not have standard long-term credentials such as a password or access keys. Instead, when an EC2 instance assumes a role, it receives temporary security credentials that it can use to make AWS API calls.

**Why Use IAM Roles over Access Keys?**

*   **Security:** You never have to store access keys in your application code or on the instance itself. Hard-coding credentials is a major security risk. If an instance is compromised, the attacker won't find credentials that they can steal and use elsewhere.
*   **Manageability:** The temporary credentials for a role are automatically rotated by AWS. You don't have to manage key rotation yourself.
*   **Granularity:** You can create roles with very specific, limited permissions (the "principle of least privilege") and attach them to specific instances. For example, one set of web servers might have a role allowing them to read from an S3 bucket, while a different set of data processing servers might have a role allowing them to write to a different bucket.

---

## 4.2 How IAM Roles for EC2 Work

The mechanism that allows an EC2 instance to assume a role is called an **Instance Profile**.

1.  **IAM Policy:** You first create a policy, which is a JSON document that explicitly lists the permissions (e.g., `Allow s3:GetObject` on `arn:aws:s3:::my-app-assets/*`).
2.  **IAM Role:** You create a role and attach the policy to it. When creating the role, you specify a "trust policy" that defines who is allowed to assume the role. For EC2, you designate the EC2 service (`ec2.amazonaws.com`) as the trusted principal.
3.  **Instance Profile:** When you create a role for EC2 using the AWS Console, an "instance profile" is automatically created with the same name. This instance profile is a container for the IAM role.
4.  **Attach to EC2:** You attach the instance profile to the EC2 instance either at launch time or while it is running.

Once attached, any application running on that instance can retrieve the temporary credentials from the instance's metadata service and use them to make secure requests to other AWS services.

---

## 4.3 Administrative Steps: Creating a Role for S3 Access

Let's walk through a common scenario: creating a role that grants read-only access to a specific S3 bucket and attaching it to our web server instances.

**Step 1: Create the IAM Policy**
1.  Navigate to the **IAM** service in the AWS Console.
2.  Go to **Policies** and click **Create policy**.
3.  Select the **JSON** tab and paste the following policy. Be sure to replace `mycompany-app-assets` with the actual name of your S3 bucket.
    ```json
    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Effect": "Allow",
                "Action": "s3:GetObject",
                "Resource": "arn:aws:s3:::mycompany-app-assets/*"
            }
        ]
    }
    ```
4.  Click **Next: Tags**, then **Next: Review**.
5.  Give the policy a name (e.g., `S3-ReadOnly-AppAssets-Policy`) and a description.
6.  Click **Create policy**.

**Step 2: Create the IAM Role**
1.  In the IAM console, go to **Roles** and click **Create role**.
2.  For "Trusted entity type", select **AWS service**.
3.  For "Use case", select **EC2** and click **Next**.
4.  In the permissions policy list, search for the policy you just created (`S3-ReadOnly-AppAssets-Policy`) and check the box next to it.
5.  Click **Next**.
6.  Give the role a name (e.g., `EC2-S3-ReadOnly-Role`) and a description.
7.  Click **Create role**.

**Step 3: Attach the Role to an EC2 Instance**
1.  Navigate to the **EC2** service in the AWS Console.
2.  Select the instance you want to grant access to.
3.  Click **Actions** -> **Security** -> **Modify IAM role**.
4.  In the "IAM role" dropdown, select the role you just created (`EC2-S3-ReadOnly-Role`).
5.  Click **Update IAM role**.

The EC2 instance now has the permissions defined in the role. Any application on that instance that uses an AWS SDK will automatically and securely fetch and use these temporary credentials to access the S3 bucket.
