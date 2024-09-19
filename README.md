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







