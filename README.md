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

Bash
# Fetch token
TOKEN=`curl -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 21600"`
# Use token to get metadata
curl -H "X-aws-ec2-metadata-token: $TOKEN" -v http://169.254.169.254/latest/meta-data/
SSM Parameter Store Fetch: The 4 lines of Python using WithDecryption=True.

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
