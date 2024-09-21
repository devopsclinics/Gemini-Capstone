VPC, Networking, and Security
Objective: Set up the networking layer, including VPC, subnets, security groups, and IAM roles.
Step-by-Step Guide:
1. Create a VPC: (or use the Default VPC)
   * Go to the VPC Dashboard in the AWS Management Console.
   * Click Create VPC.
   * Choose VPC with a Single Public Subnet to keep it simple.
   * Set CIDR Block to 10.0.0.0/16 and name VPC (e.g., "DatingSite-VPC").
   * Enable DNS Hostnames so your EC2 instances can resolve DNS names.
   * Click Create VPC.
2. Create Public and Private Subnets:
   * In the VPC Dashboard, go to Subnets > Create Subnet.
   * Select your VPC (e.g., "DatingSite-VPC").
   * Create two public subnets:
      * PublicSubnetA (10.0.0.0/24 in Availability Zone A).
      * PublicSubnetB (10.0.1.0/24 in Availability Zone B).
   * Select the public subnet (both public subnets), click “action” and select “edit subnet settings”. Then check the box for “Enable auto-assign public IPv4 address.
   * Create two private subnets:
      * PrivateSubnetA (10.0.16.0/20 in Availability Zone A).
      * Private Subnet 2 (10.0.32.0/24 in Availability Zone B).
3. Attach an Internet Gateway:
   * Go to the VPC Dashboard > Internet Gateways > Create Internet Gateway.
   * Name it ("DatingSite-IGW") and click Create.
   * After creation, attach the gateway to your VPC (DatingSite-VPC).
4. Route Tables Configuration:
   * Go to Route Tables > Create Route Table.
   * Create a Public Route Table:
      * Name it ("Public-RT").
      * Associate the public subnets.
      * Edit the routes and add a route for all outbound traffic (0.0.0.0/0) pointing to the Internet Gateway.
   * Create a Private Route Table:
      * Name it (e.g., "Private-RT").
      * Associate the private subnets.
5. NAT Gateway:
   * In the VPC Dashboard, go to NAT Gateways > Create NAT Gateway.
   * Place the NAT Gateway in one of the public subnets.
   * Associate the Private Route Table with the NAT Gateway for outbound internet traffic.
6. Security Groups:
   * Go to EC2 Dashboard > Security Groups > Create Security Group.
   * ELB Security Group:
      * Allow inbound traffic on ports 80/443 from the internet.
   * EC2 Security Group:
      * Allow inbound traffic on port 80 (HTTP) and 443 (HTTPS) from the ELB Security Group.
      * Allow outbound traffic to RDS on port 3306 (MySQL) and Elasticache (port 6379).
   * RDS Security Group:
      * Allow traffic from EC2 instances and Bastion Host on port 3306.
   *  Bastion Host Security Group:
      * Allow SSH (port 22) from your IP.
      * Allow traffic to RDS instances in the private subnet on port 3306.
   * Elasticache Security Group:
      * Allow inbound traffic from EC2 instances.
7.   Create a Bastion Host:
   * Launch an EC2 Instance in a PublicSubnetA using the Amazon Linux 2 AMI.
   * Choose t2.micro for cost-effectiveness (Free Tier eligible).
   * Attach a Security Group that allows SSH access (port 22) from your IP only (for secure access).
   * Attach an Elastic IP to the Bastion Host, so you can connect to it consistently.
   * The Bastion Host allows you to connect securely to RDS and EC2 instances in the private subnets.
8. IAM Role Setup:
   * Go to the IAM Dashboard > Roles > Create Role.
   * EC2 Role:
      * Choose AWS service and select EC2.
      * Attach AmazonS3ReadOnlyAccess policy to allow EC2 instances access to S3.
      * Name the role ( "EC2-S3Access-Role").
   * Lambda Role:
      * Choose AWS service and select Lambda.
      * Attach policies like AmazonRDSFullAccess and AWSLambdaExecute.

Step-by-step guide to set up EC2 Instances, Auto-Scaling, and an Application Load Balancer (ALB)

Set up EC2 Instances:
In the AWS Management Console search bar at the top, type "EC2" and select EC2.
Select Amazon Linux 2 AMI or another suitable AMI for your app.
Select t2.micro (Free Tier eligible).

Network Settings:
 Select your VPC created earlier, and choose a private subnet.
Auto-assign Public IP: Disable this setting, as the instance is in a private subnet.
Key Pair: Choose an existing key pair or create a new one.

Set Up Auto-Scaling Group (ASG):
In the EC2 Dashboard, scroll down to Auto Scaling Groups.
Click Create Auto Scaling Group.
Configure your ASG by naming your group and choose a Launch Template. If you haven’t created a Launch Template yet:

Create a Launch Template specifying the instance type (eg, choose the Amazon Linux 2 AMI, t2.micro), and select the security group and key pair.
VPC and Subnet: Select your VPC and choose private subnets configured earlier across two or more Availability Zones (AZs).
If you already have an ASG: Select your existing Auto Scaling Group, click Actions, and choose Edit to modify the scaling policies.

Set Target Tracking Scaling Policy:
In the Scaling Policies section:
Select Target tracking scaling policy.
Set the Metric Type to Average CPU Utilization.
Set the target value (CPU utilization) to 70%.
            Add Scaling Adjustment:
Set the scale-down threshold: To save costs, configure the Auto Scaling Group to remove instances when CPU utilization drops below 40%.
Cooldown Period:
Set an appropriate cooldown period (e.g., 300 seconds) to ensure that the scaling actions have time to take effect before further scaling adjustments are triggered.
           Click Save



Set up Application Load Balancer (ALB):
In the search bar, type "EC2" and click on EC2 under Services.
On the left menu, under Load Balancing, click Load Balancers.
On the Load Balancers page, click Create Load Balancer.

Choose Load Balancer Type:
Select Application Load Balancer for HTTP/HTTPS traffic.
Configure Basic Settings:
Name: Give your ALB a meaningful name (e.g., dating-website-alb).
Scheme: Set it to Internet-facing to allow access from the internet.
IP Address Type: Choose IPv4 (or Dualstack if using both IPv4 and IPv6).

Configure Network Settings
VPC: Select the VPC where your EC2 instances are running.
Availability Zones:
Select the public subnets for each Availability Zones.
Subnet Configuration:
 Choose Public Subnets (in different AZs) to allow traffic from the internet.

Create Target Group: to route traffic to your EC2 instances in the private subnet.
Choose Instance as the target type (since you want to route traffic to EC2 instances).
VPC: Select the same VPC where your EC2 instances are located.
Health Check Protocol: Set the Health Check Protocol to HTTP or TCP depending on your application's configuration.
For HTTP, set the Health Check Path to something like /health if your application exposes a health check endpoint.

Other Health Check Settings: Configure the Healthy Threshold (e.g., 3 successful checks) and Unhealthy Threshold (e.g., 2 failed checks) 
Set Health Check Interval (e.g., 30 seconds) 
Register EC2 Instances with Target Group
Select the EC2 instances in the private subnets from the list.
Click Include as Pending Below, then click Register Targets.

Review all of your configurations:
Ensure the ALB is placed in public subnets and will distribute traffic to EC2 instances in private subnets via the Target Group.
Click Create Load Balancer.
Verify Load Balancer and Health Checks
Go to Load Balancers in the EC2 Dashboard to monitor the status of the ALB.
Go to Target Groups:
In the Target Group section, check if your registered EC2 instances are marked as healthy.
If instances are marked as unhealthy, verify that your health check configuration (path, interval, etc.) is correct and accessible by the ALB.
Add Listener Rules for Specific Traffic (optional):
You can add Listener Rules to forward certain types of traffic:
eg, forward requests for /api/* to EC2 instances (dynamic content), while routing static content requests to CloudFront


Set Up ElastiCache for Session Management (Redis)
In the search bar of the AWS Management Console, type “ElastiCache” and click on ElastiCache under Services.
Create a Redis Cluster
Click “Create” in the ElastiCache dashboard.
Select Redis under the Cluster Engine.
Name your cluster (e.g., redis-cluster).


Configure Redis Cluster
Cluster Mode: Choose Cluster Mode Disabled (if you're using a simple Redis setup) or Cluster Mode Enabled (for advanced, distributed Redis clusters).
Node Type: Choose a node type (e.g., cache.t2.micro for development and testing, or a larger instance for production).
Number of Nodes:
Primary Node: Set one primary node.
Replica Nodes: Add one or more replica nodes across different Availability Zones to ensure high availability. This way, if the primary node fails, a replica node will take over.
Multi-AZ: Enable Multi-AZ with Automatic Failover to ensure that if the primary node goes down, a replica in another AZ will take over.
VPC and Subnet Group:
Select the VPC where your EC2 instances are running.
Choose or create a Subnet Group that spans multiple AZs to ensure high availability.


Configure Security Settings
Security Group: Choose or create a security group that allows inbound traffic from your EC2 instances (the ones that need to communicate with Redis).
Encryption:
Enable in-transit encryption and at-rest encryption to secure your data.
Set authentication if needed (enforce Redis AUTH for more security).
Finalize and Launch


Set Up Session Management on EC2
In the ElastiCache dashboard, locate your Redis cluster.
Click on your Redis cluster to open its details.
Find the Redis Endpoint:
In the Cluster Details section, locate the Primary Endpoint URL to connect to the Redis cluster.

Modify Your Application on EC2 to use Redis:
SSH into your EC2 instance where your web application is running
ssh -i /path/to/keypair.pem ec2-user@ec2-instance-public-ip
In the application code, add the Redis connection configuration, and use the Redis cluster's endpoint URL (provided in the ElastiCache dashboard) to connect to Redis.

Example for a Node.js app using Redis:
const redis = require('redis');
const client = redis.createClient({
    host: 'your-redis-endpoint-url',
    port: 6379, // Default Redis port
    password: 'your-redis-password', // If Redis AUTH is enabled
});

// Set session data
client.set('sessionID', 'sessionData', redis.print);

// Get session data
client.get('sessionID', (err, reply) => {
    console.log(reply); // prints sessionData
});
Install Redis Libraries:
           If your application doesn’t have Redis libraries installed, install them. For Node.js, you can install it like this:
            npm install redis


Test the Setup:
Deploy your application on your EC2 instances and confirm that session data is stored in Redis by testing the functionality of your web application.
Verify that session data is persistent even when traffic is distributed across different EC2 instances via the load balancer.

On the EC2 instance, use the Redis CLI to check the stored session data:
redis-cli -h <your-redis-endpoint-url> -p 6379
                              Use the get command to check if session data is stored:
                                   get sessionID


Set Up CloudFront (Content Delivery Network)
In the search bar of AWS Management Console, type “CloudFront” and click on CloudFront under Services.
On the CloudFront dashboard, click Create Distribution.
Choose Web Distribution for your static content (e.g., images, CSS, and JavaScript files).


Configure Origin Settings (S3 Integration)
Origin Domain Name:
Select your S3 bucket where your static files (e.g., profile images, CSS, JavaScript) are stored.
If your S3 bucket is not public, configure S3 bucket policies to allow CloudFront to access the content through:
Origin Access Identity (OAI) to securely access the private content in your S3 bucket.


Create or Modify the S3 Bucket Policy:
Below is an example policy to allow CloudFront to read objects from the S3 bucket:
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::cloudfront:user/CloudFront Origin Access Identity OAI_ID"
      },
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::your-bucket-name/*"
    }
  ]
}
Replace OAI_ID with your CloudFront OAI ID.
Replace your-bucket-name with your actual S3 bucket name.

Origin Path: Specify the folder (if applicable) within the S3 bucket where your static assets are stored (e.g., /static).
Restrict Bucket Access: Enable Restrict Bucket Access if you want to serve files only through CloudFront, not directly from S3.

Configure Default Cache Behavior
Viewer Protocol Policy:
Set this to Redirect HTTP to HTTPS to ensure secure connections.
Allowed HTTP Methods: Choose GET, HEAD (for serving static content).
Object Caching: Set caching behavior. You can use origin cache headers or customize the TTL (Time-to-Live) for cached objects.

Configure Distribution Settings
Price Class: Choose a Price Class based on the geographical locations where you expect most traffic. For global access, choose Use All Edge Locations.
SSL Certificate:
If you have an SSL certificate, choose Custom SSL Certificate and configure it.
If not, use the default CloudFront certificate (for https://domain.cloudfront.net).

Create the Distribution
Review the settings and click Create Distribution. It may take a few minutes for the distribution to be deployed.
Once deployed, CloudFront will begin caching and serving content from the edge locations closest to users.

Route Static Content Requests via CloudFront
Update Your DNS (Route 53):
In Route 53, create a CNAME record (e.g., static.yourwebsite.com) that points to the CloudFront distribution's domain name (e.g., abc123.cloudfront.net).
Update Your Application:
Modify your application to serve static content (e.g., images, CSS) via the CloudFront URL (e.g., https://static.yourwebsite.com/path-to-image).


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

	Lambda Functions
1. Set Up the S3 Bucket for Photo Uploads
•	Create an S3 bucket where users will upload their photos.
•	Go to S3 in the AWS Management Console.
•	Click Create Bucket, name the bucket and configure permissions.
2. Set Up AWS Lambda for Image Processing
Create a Lambda Function:
•	Go to AWS Lambda in the AWS Management Console.
•	Click Create Function.
•	Choose Author from scratch and provide a function name- ImageProcessorFunction
•	Select Node.js, Python or preferred language.
•	Choose an existing role or create a new one with S3 and CloudWatch permissions.
Write the Lambda Function: The function will resize or optimize images after being uploaded to S3. Here's Node.js that uses the sharp library to resize an image:

	const AWS = require('aws-sdk');
	const S3 = new AWS.S3();
	const Sharp = require('sharp');
	exports.handler = async (event) => {
    	const bucket = event.Records[0].s3.bucket.name;
    	const key = decodeURIComponent(event.Records[0].s3.object.key.replace(/\+/g, ' '));
    try {
        // Get the uploaded image from S3
        const image = await S3.getObject({ Bucket: bucket, Key: key }).promise();
        
        // Resize the image using Sharp
        const resizedImage = await Sharp(image.Body)
            .resize(800, 600)  // Resize to 800x600
            .toBuffer();
        
        // Upload the resized image back to S3
        await S3.putObject({
            Bucket: bucket,
            Key: `resized/${key}`,  // Store the image in a resized folder
            Body: resizedImage,
            ContentType: 'image/jpeg'
        }).promise();
        
        return {
            statusCode: 200,
            body: JSON.stringify('Image successfully resized and uploaded.'),
        };
    } catch (error) {
        console.error(error);
        return {
            statusCode: 500,
            body: JSON.stringify('Error processing image.'),
          };
    	}
    };


3. Set Up the S3 Event Trigger
Configure S3 to trigger the Lambda function when a new image is uploaded:
•	Go to your S3 bucket in the AWS Management Console.
•	Navigate to Properties > Event Notifications.
•	Add a new event notification:
•	Event name: ImageUploadEvent.
•	Event type: Choose PUT (for new object creation).
•	Prefix: You can set a folder (e.g., /uploads/), so only images uploaded to this folder trigger the function.
•	Lambda function: Select your Lambda function (ImageProcessorFunction).
To implement AWS Lambda for serverless tasks such as SNS-based notifications (e.g., user alerts or status updates) on your platform, you can follow these steps:
1. Create an SNS (Simple Notification Service) Topic
o	Go to Amazon SNS in the AWS Management Console.
o	Create an SNS Topic
o	In the SNS Dashboard, click Create Topic.
o	Choose a topic type: Standard (supports high-throughput, best-effort message delivery) or FIFO (for order-sensitive notifications).
o	Name your topic- UserAlertsTopic
o	After creating the topic, you'll get the Topic ARN (Amazon Resource Name) which will be used in Lambda.
         Add Subscriptions:
•	Click on the newly created topic.
•	Add a subscription by clicking Create Subscription.
•	Choose a protocol (e.g., Email, SMS, HTTP/S, etc.).
•	Enter the subscription endpoint (e.g., an email address or phone number).
•	Confirm the subscription by checking your inbox or mobile device for a confirmation message from AWS SNS.
2. Create the Lambda Function for SNS Notifications
Create a Lambda Function:
Go to AWS Lambda in the AWS Management Console.
Click Create Function.
Choose Author from scratch and provide a function name (e.g., SendUserAlert).
Choose a runtime such as Node.js, Python, or any preferred language.
Choose or create an execution role that has permissions for SNS Publish and CloudWatch Logs.
Write the Lambda Function:
	
	const AWS = require('aws-sdk');
	const sns = new AWS.SNS();

	exports.handler = async (event) => {
    const params = {
        Message: `Hello, user! Your status is now: ${event.statusUpdate}`, // Customize the message
        TopicArn: 'arn:aws:sns:region:account-id:UserAlertsTopic'  // Replace with your SNS topic ARN
    };
    
    try {
        const result = await sns.publish(params).promise();
        console.log(`Message sent: ${result.MessageId}`);
        
        return {
            statusCode: 200,
            body: JSON.stringify('Notification sent successfully'),
        };
    } catch (error) {
        console.error('Error sending message:', error);
        return {
            statusCode: 500,
            body: JSON.stringify('Failed to send notification'),
        };
       }
    };
NOTE: Message: Customize the message to include any dynamic data, such as a user’s name, profile status, or system event.
TopicArn: Replace this with your actual SNS Topic ARN.
2. Set Up Lambda Triggers
You can trigger the Lambda function based on various events such as user actions, system events, or alarms from other AWS services (e.g., CloudWatch).
Example: Trigger Lambda from CloudWatch Alarm 
If you want Lambda to send notifications based on a CloudWatch alarm or a database change (e.g., when a user profile is updated):
1.	For CloudWatch Alarm:
o	Create a CloudWatch Alarm for any metric (e.g., high CPU utilization).
o	In the alarm's Actions, select Trigger a Lambda function and choose your SendUserAlert function.

	CloudWatch Monitoring
1.	Enable EC2 Monitoring:
By default, basic monitoring (with 5-minute intervals) is enabled for EC2 instances. To enable detailed monitoring (with 1-minute intervals):
Go to the EC2 Dashboard.
Select an instance.
Under the Monitoring tab, click Manage Detailed Monitoring.
2.	View EC2 Metrics in CloudWatch:
Go to CloudWatch in the AWS Management Console.
In the left sidebar, click Metrics > EC2 to view the available instance metrics, such as:
CPUUtilization
NetworkIn/Out
DiskReadOps/WriteOps
3.	Create an Alarm for EC2 Auto-Scaling:
To set an alarm that triggers auto-scaling based on resource usage (e.g., CPU utilization exceeding 80%):
In CloudWatch, go to Alarms and click Create Alarm.
Select EC2 metrics and choose the metric, e.g., CPUUtilization.
Set the threshold (e.g., 80%) and choose greater than or equal to.
Configure an action: Choose Auto Scaling Group and set it to scale up or down when the alarm is triggered.

4.	Set Up Alarms for EC2 Failure Detection:
You can create an alarm for failure detection, such as monitoring when an EC2 instance goes into a stopped or unhealthy state.
Select InstanceStatusCheckFailed or SystemStatusCheckFailed as the metric.
Set the threshold to 1 and configure a notification action (e.g., send an alert via SNS or email).
CloudWatch Monitoring for Lambda Functions
1.	Enable Monitoring for Lambda Functions:
Lambda functions automatically generate CloudWatch logs, but you can also view additional metrics such as:
Invocations: Number of times the function was invoked.
Errors: Number of failed function executions.
Duration: How long the function runs per execution.
Throttles: Number of times execution was throttled due to concurrency limits.

2.	View Lambda Metrics:
Go to CloudWatch in the AWS Management Console.
In the left sidebar, go to Metrics > Lambda.
You will see metrics for all Lambda functions, including:
Duration
Invocations
Error Count
Throttles
3.	Create Alarms for Lambda:
Performance Monitoring: Create an alarm for function duration if it exceeds a certain time (e.g., if execution time exceeds 500ms).
Go to Alarms and click Create Alarm.
Select Lambda and choose the Duration metric.
Set the threshold (e.g., greater than 500ms) and configure a notification action.
Failure Detection: Set an alarm for errors or failed invocations.
Choose the Errors or Throttles metric.
Set the threshold (e.g., greater than or equal to 1) and configure an SNS notification to alert you when errors occur.

CloudWatch Monitoring for Other AWS Resources
1.	For RDS (Relational Database Service):
Monitor key database metrics such as CPU usage, memory, disk I/O, and free storage space.
Navigate to CloudWatch > Metrics > RDS.
Select metrics like CPUUtilization, FreeStorageSpace, DatabaseConnections, etc.
Create alarms to alert you if the database is under high load or running out of space.
2.	For S3 (Simple Storage Service):
Monitor S3 metrics such as NumberOfObjects and BucketSizeBytes.
Navigate to CloudWatch > Metrics > S3.
Set up alarms for large bucket sizes or excessive object counts.
3.	For DynamoDB:
Monitor throughput (read/write capacity), latency, throttled requests, and item count.
Navigate to CloudWatch > Metrics > DynamoDB.
Create alarms for ConsumedReadCapacityUnits or ConsumedWriteCapacityUnits to prevent over-usage of provisioned capacity.

Set Up CloudWatch Logs for All Services
1.	Create Log Groups:
For EC2, install and configure the CloudWatch Logs Agent to push system logs to CloudWatch.
For Lambda, logs are automatically created under the function’s name.

2.	Set Log Retention and Monitoring:
Go to Logs in the CloudWatch dashboard.
Set log retention policies (e.g., 7 days, 30 days, etc.).
You can create metric filters to detect specific patterns in logs (e.g., error messages) and trigger alarms.
Creating Alarms for Auto-Scaling, Performance, and Failure Detection
1.	Auto-Scaling Alarms:
For EC2, you can create auto-scaling alarms that increase/decrease instance counts based on CPU, memory, or custom metrics.
For Lambda, use alarms to control concurrency limits or trigger actions when invocation counts or durations spike.
2.	Performance Monitoring:
Set alarms to monitor performance across all services (e.g., high latency in RDS, Lambda duration, etc.).
Use a combination of metrics to detect bottlenecks (e.g., CPU spikes on EC2 or read/write throttling in DynamoDB).
3.	Failure Detection:
For EC2: Use SystemStatusCheckFailed or InstanceStatusCheckFailed metrics.
For Lambda: Set alarms for Errors or Throttles.
For S3 or RDS: Set up alarms for failure scenarios like unresponsive endpoints, connection errors, or capacity overuse.

	SNS for Notifications
Amazon SNS Topics allow you to group multiple subscribers under a single notification event. For example, you can create separate topics for user notifications, admin notifications, and downtime alerts. To create SNS Topic:
1.	Go to the SNS Dashboard:
Navigate to the SNS service in the AWS Management Console.
2.	Create a Topic:
Click on Create topic.
Choose a topic type:
Standard: Best for high-throughput notifications.
FIFO (First-In-First-Out): Useful if notification order is important (e.g., transaction logs).
Name the Topic (e.g., UserNotificationTopic for user alerts or AdminAlertTopic for admin notifications).
After creating the topic, note down the Topic ARN. This will be used later for publishing messages.

Add Subscribers to the Topic
Subscribers can receive notifications via different channels, such as Email, SMS, Mobile Push, HTTP/S endpoints, or AWS Lambda.
1.	Go to the Topic:
After creating a topic, click on the topic name to configure it.
2.	Create Subscriptions
•	Click on Create subscription.
•	Choose a protocol: Select the notification delivery method, such as:
o	Email: Alerts will be sent to users’ or admins’ email addresses.
o	SMS: Messages will be sent via text.
o	HTTP/HTTPS: Notifications can be sent to an HTTP/S endpoint (such as a webhook).
o	Lambda: Trigger AWS Lambda functions for more advanced workflows.
•	Enter the subscription endpoint (e.g., an email address or mobile number).
•	Click Create subscription. For email or SMS, the user or admin will receive a confirmation message to confirm their subscription.

3.	Add Multiple Subscribers:
For topics like AdminAlertTopic, you can add multiple email addresses (for different administrators) or phone numbers (for text alerts).
Users and admins can unsubscribe at any time by following the instructions in the notification emails or texts.
Configure Notifications for Different Events
You can configure SNS to send notifications for different application events like status changes, new matches, messages, or downtime alerts. These notifications will be triggered using Lambda functions, CloudWatch alarms, or your application backend.
Common Event Scenarios:
•	Application Status Changes (e.g., profile approval or rejection):
•	When a user’s profile is approved or rejected, an SNS notification is sent to the user.
•	New Matches: When two users match on the platform, send notifications via SNS.
•	New Messages: Notify users via email or SMS when they receive new messages.
•	Downtime Alerts: Notify admins when the application experiences downtime.


SNS Setup for Event-Based Notifications: 
1.	Lambda Function to Publish SNS Messages:
•	You can use AWS Lambda to trigger SNS messages whenever specific application events occur. For example, when a new match is found, or a user gets a message, invoke Lambda to publish an SNS message.
Node.js Lambda function to send an SNS notification for a new match:
	const AWS = require('aws-sdk');
	const sns = new AWS.SNS();

	exports.handler = async (event) => {
    const params = {
        Message: `Congrats! You've got a new match with ${event.matchName}!`,
        TopicArn: 'arn:aws:sns:region:account-id:UserNotificationTopic'
    };
    
    try {
        const result = await sns.publish(params).promise();
        console.log(`Message sent: ${result.MessageId}`);
        return { statusCode: 200, body: 'Notification sent successfully.' };
    } catch (error) {
        console.error('Error sending message:', error);
        return { statusCode: 500, body: 'Failed to send notification.' };
     }
    };
2.	CloudWatch Alarms for Downtime Alerts:
Set up CloudWatch alarms to monitor critical resources (EC2, RDS, Lambda, etc.).
When a failure or downtime is detected (e.g., CPU utilization spike, failed status checks), the CloudWatch alarm triggers SNS to notify administrators.
Go to CloudWatch Alarms.
Create an alarm (e.g., monitor EC2 instance health).
Under Actions, select Send a notification to an SNS topic.
Choose the AdminAlertTopic you created earlier.







