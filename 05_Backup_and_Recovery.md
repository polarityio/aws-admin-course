# 5. Data Backup and Recovery

A critical responsibility for any administrator is ensuring that data can be recovered in the event of an application failure, accidental deletion, or instance corruption. For EC2 instances, the primary method of backup is creating snapshots of their Elastic Block Store (EBS) volumes.

---

## 5.1 Understanding EBS Snapshots

An [EBS Snapshot](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EBSSnapshots.html) is a point-in-time backup of an EBS volume. The snapshot is stored independently from the volume itself in Amazon S3, which provides high durability.

**Key Characteristics:**

*   **Point-in-Time:** The snapshot captures the exact state of the volume at the moment the snapshot command is initiated.
*   **Incremental:** The first snapshot you take of a volume copies all the data. For subsequent snapshots of the same volume, only the blocks that have changed since the last snapshot are saved. This minimizes the time required to create the snapshot and saves on storage costs.
*   **Crash-Consistent:** Snapshots automatically capture everything on the volume. For volumes attached to a running instance, this is similar to suddenly pulling the power cord—the operating system is not given a chance to flush all its caches to disk. For most workloads, this is sufficient. For databases, it is highly recommended to stop the instance or detach the volume before taking a snapshot to ensure a perfectly consistent backup.

---

## 5.2 Administrative Steps: Manual Snapshot and Restore

### Creating a Manual Snapshot
1.  Navigate to the **EC2** service in the AWS Console.
2.  Under "Elastic Block Store", select **Volumes**.
3.  Select the EBS volume that is attached to your EC2 instance.
4.  Click **Actions** -> **Create snapshot**.
5.  Provide a clear and descriptive name for the snapshot (e.g., `WebServer1-Pre-Update-Backup-2025-08-22`). Adding tags, like the instance ID and date, is also a best practice.
6.  Click **Create snapshot**. The snapshot will begin processing in the background.

### Restoring a Volume from a Snapshot
If your original volume fails or the data becomes corrupt, you can create a new, identical volume from your snapshot.

1.  In the EC2 console, go to **Snapshots**.
2.  Select the snapshot you wish to restore from.
3.  Click **Actions** -> **Create volume from snapshot**.
4.  Configure the new volume. **Crucially, you must create the new volume in the same Availability Zone as the instance you want to attach it to.**
5.  Click **Create volume**.
6.  Once the new volume is created, you can detach the old, failed volume from your EC2 instance and attach the newly restored volume in its place.

---

## 5.3 Automating Backups with Amazon Data Lifecycle Manager

Manually creating snapshots is not a scalable or reliable long-term strategy. The best practice is to automate the process using the **Amazon Data Lifecycle Manager**. This service allows you to create "lifecycle policies" that define a schedule for snapshot creation and retention.

**Administrative Steps: Creating a Lifecycle Policy**
1.  In the EC2 console, go to **Lifecycle Manager** under "Elastic Block Store".
2.  Click **Create lifecycle policy**.
3.  **Policy Type:** Choose **EBS snapshot policy**.
4.  **Target Resources:** You can target volumes by tags. For example, you could add a tag `Backup: True` to all your critical EC2 instances' volumes. Then, you would specify that tag here. This is a powerful way to ensure all important volumes are automatically backed up.
5.  **Description:** Give the policy a descriptive name.
6.  **IAM Role:** Select the **Default** role.
7.  **Schedule:** Define when you want snapshots to be created. For example, you could create a schedule named "Daily" that runs every 24 hours at a specific time (e.g., during off-peak hours).
8.  **Retention Rule:** This is the most important part. You define how long you want to keep the snapshots. A common strategy is a tiered approach:
    *   Keep **7** daily snapshots (by setting "Count" to 7).
    *   You could add another schedule to keep weekly or monthly snapshots for longer-term retention.
9.  Review the policy and click **Create policy**.

Once created, this policy will run automatically, ensuring you always have recent, regular backups of your critical data without any manual intervention.
