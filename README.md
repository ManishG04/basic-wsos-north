## The Golden Build Sequence (Order of Operations)
When you look at the diagram tomorrow, ignore the arrows for a minute. You must build from the bottom up.

Step 1: The Network Layer (The Foundation)
You cannot put a database or a Lambda function in a secure network if the network doesn't exist yet.

- 1st: Create the VPC and Subnets (if they aren't provided).

- 2nd: Create VPC Endpoints (for S3, DynamoDB, or Secrets Manager if your compute is private).

- 3rd: Create empty Security Groups. (Create lambda-sg, rds-sg, ec2-sg now. You can fill in the Inbound/Outbound rules later, but you need them to exist so you can reference them!)

Step 2: The Data & Storage Layer (The Stateful Tier)
Compute needs somewhere to read and write. Build these next so you can copy their ARNs and URLs.

- 1st: Create S3 Buckets (Enable Versioning, configure BPA, add Lifecycle rules).

- 2nd: Create DynamoDB Tables (Set partition keys, enable Target Tracking Auto Scaling).

- 3rd: Create RDS Databases (Put them in private subnets, disable public access).

Step 3: The Security & Configuration Layer (The Vault)
Before you write a single line of code, you need the permissions and the secrets ready.

- 1st: Create KMS Customer Managed Keys (CMKs).

- 2nd: Store database passwords in Secrets Manager or API keys in SSM Parameter Store. (Encrypt them with your KMS keys).

- 3rd: Create your IAM Execution Roles. (e.g., Create the Lambda role and attach the policies allowing it to read the Secrets, decrypt the KMS key, and write to DynamoDB).

Step 4: The Compute Layer (The Engine)
Now you build the brains, because you have all the pieces ready to plug into it.

- 1st: Create the Lambda Functions.

- 2nd: Attach the IAM Execution Role you made in Step 3.

- 3rd: Attach the VPC and Security Groups you made in Step 1.

- 4th: Add Environment Variables (paste the DynamoDB table names or S3 bucket names here).

Step 5: The Decoupling Layer (The Pipes)
Now you connect the storage and compute together.

- 1st: Create SQS Queues (and their DLQs).

- 2nd: Create SNS Topics (and subscribe the SQS queues to them).

- 3rd: Create EventBridge Rules (and point them to your Lambda or SQS targets using the IAM Execution Role).

Step 6: The Front Door (The Entrypoint)
Finally, you expose the architecture to the judges or the users.

- 1st: Create the API Gateway (Point it to your Lambda functions, enable CORS, enable caching).

- 2nd: Create the Application Load Balancer (ALB) or CloudFront Distribution.


## The "Invisible Diagram" Traps
Architecture diagrams in competitions are intentionally incomplete. They show you the data flow, but they hide the security and networking glue. When you look at the diagram tomorrow, you must visualize the invisible lines.

1. The Invisible IAM Line:

What the diagram shows: An arrow pointing from S3 to Lambda to DynamoDB.

What you must build: You must manually go to IAM and give Lambda a role with s3:GetObject and dynamodb:PutItem. The diagram won't tell you to do this; it's expected knowledge.

2. The Invisible Security Group Line:

What the diagram shows: An arrow from an EC2 instance to an RDS database.

What you must build: You must go to the RDS Security Group and add an Inbound Rule allowing Port 3306/5432 from the EC2 Security Group.

3. The Invisible "VPC Loss of Internet" Line:

What the diagram shows: Lambda inside a VPC talking to RDS, and also talking to an external API.

What you must build: You must provision a NAT Gateway in a public subnet, or the Lambda will time out trying to reach the internet.


## Extra stuff
1. The "Fault Finding" Matrix (The Holy Grail)
This is for when you are staring at a broken architecture and don't know where to start. Format it as "Symptom -> Culprit -> Fix".

Symptom: Lambda times out connecting to RDS.

Fix: Lambda is missing VPC attachment, OR Security Group on RDS is blocking Lambda's SG.

Symptom: EC2 instance constantly terminates and recreates.

Fix: ASG Health Check Grace Period is too short, OR ALB is pinging the wrong health check path (e.g., /index.html instead of /).

Symptom: API Gateway works in Postman but fails in the browser.

Fix: Enable CORS on the API Gateway resource and deploy the API.

Symptom: Lambda is failing but there are no CloudWatch logs.

Fix: Lambda Execution Role is missing logs:CreateLogGroup and logs:PutLogEvents.

Symptom: SQS messages are being processed twice.

Fix: SQS Visibility Timeout is shorter than the Lambda execution time. Increase the SQS Visibility Timeout.

2. The JSON Policy Bank
Under pressure, you will make a JSON syntax error (missing a comma or a bracket). Have these ready to copy, paste, and modify:

The S3 Bucket Policy for VPC Endpoints: (To enforce that S3 is only accessed from your private VPC).

The KMS Key Policy: (Allowing Lambda to use the key, and allowing S3/CloudTrail to use the key).

Cross-Account IAM Role Trust Policy: (The sts:AssumeRole JSON).

S3 Block Non-HTTPS Traffic: (The aws:SecureTransport: false deny policy we discussed).

3. Boto3 & CLI Snippets
Since there is minimal coding, you don't need full scripts, but you do need the exact commands to test things or fetch secrets.

EC2 IMDSv2 Command: The exact curl command with the session token to fetch EC2 metadata (since they explicitly mentioned metadata).

```Bash
# Fetch token
TOKEN=`curl -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 21600"`
# Use token to get metadata
curl -H "X-aws-ec2-metadata-token: $TOKEN" -v http://169.254.169.254/latest/meta-data/
SSM Parameter Store Fetch: The 4 lines of Python using WithDecryption=True.
```

Secrets Manager Fetch: The get_secret_value syntax.

4. The Well-Architected Cheat Sheet (By Category)
Since you are graded on these exact pillars, have a checklist to run through before you submit a task to ensure you squeezed every point out of it:

Security (22): Are KMS keys rotated? Is S3 BPA on? Are Security Groups referencing other SGs instead of IPs? Are API Keys required on API Gateway?

Cost Optimization (8): Is S3 Intelligent Tiering used? Is DynamoDB on Provisioned capacity with Target Tracking? Are old EBS snapshots being deleted?

Reliability (19): Is RDS Multi-AZ enabled? Does SQS have a DLQ attached? Is S3 Cross-Region Replication turned on with Versioning?

Performance (11): Is API Gateway caching enabled? Does Lambda have Provisioned Concurrency? Is S3 Transfer Acceleration on?

Operational Excellence (22): Are X-Ray traces enabled? Are CloudWatch Alarms hooked up to SNS? Is CloudTrail data event logging on?

Sustainability (10): Did I use ARM-based Graviton instances (t4g instead of t3) where possible? Did I set an S3 lifecycle rule to delete old versions?

Work Management (8): Are all resources Tagged?

5. Amazon Q Prompt Templates
Since you have access to Amazon Q, keep pre-written, highly specific prompts ready to copy-paste.

"I am getting an [INSERT ERROR CODE] in CloudWatch for my Lambda function trying to reach [INSERT SERVICE]. What specific IAM permissions or VPC Endpoint configurations are missing?"

"How do I configure an EventBridge rule to catch an S3 object upload, but only if the file ends in .pdf, and format the output for an SQS queue?"

"What is the most cost-effective and sustainable way to architecture [INSERT REQUIREMENT] using AWS serverless services?"
