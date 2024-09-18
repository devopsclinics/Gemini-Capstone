Step-by-Step Guide to Setting Up S3 for Storing Static Assets
 
Objective: Set up S3 for static files and RDS for the database.

Step-by-Step Guide

Navigate to Amazon S3

Click on Create Bucket.

Name the bucket (e.g., "dating-site-static-content").  Naming S3 bucket should follow the following rules:

No uppercase and underscore
Must be 3-63 characters
Must not be an IP
Start with lowercase or number
Must not start with prefix --xn
Must not end in suffix --s3alias

Disable public access by default.

Enable versioning and encryption (SSE-S3 or SSE-KMS).

Set up a bucket policy to allow CloudFront access to serve content:

Go to the CloudFront Console.
Click Create Distribution.
Set your S3 bucket as the origin for the distribution.
Under Origin settings, enable Origin Access Control (OAC)
When creating the distribution, CloudFront will ask if you want to create a new OAC. Choose create new
CloudFront automatically generates an OAC that is associated with the distribution.
When you enable OAC, CloudFront automatically manages the bucket policy to grant access.
No additional manual setup is needed in the S3 bucket policy. AWS will automatically configure the permissions to allow CloudFront to access the S3 bucket.

Set up RDS (MySQL):

Go to the RDS Dashboard 
Create a Database.
Choose MySQL and Free Tier eligible settings.
Enable Multi-AZ deployment for high availability.
Place the RDS in a private subnet.

Set automatic backups and enable encryption:

            Steps to Set Up Automated Backups Using AWS Backup:

Navigate to the AWS Backup Console.
Create a Backup Plan:
Click on Backup plans and choose Create backup plan.
Choose either a predefined template or Build a new plan.
Configure Backup Plan Settings:

Backup Rule Name: Name your backup rule (e.g., DailyRDSBackup).
Backup Frequency: Set the frequency (e.g., daily, weekly).
Backup Window: Define when the backup should be performed (e.g., during off-peak hours).
Retention Period: Choose the retention period (e.g., 7 days, 30 days, etc.).
Assign Resources: In the Resource assignments section, click Assign resources.
Provide a name (e.g., RDSBackupResources) and select Amazon RDS from the list of services.
Choose the specific RDS instance(s) you want to back up.

Backup Monitoring: After the backup plan is created, you can monitor backup jobs under the Jobs section of the AWS Backup console.

Attach the RDS Security Group to limit access to EC2.





