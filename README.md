# Elastic-Spektra


## Deployment

The solution can be deployed to customer member accounts which are part of the AWS Organization so it will collect logs accordingly. We have considered the following inputs for log ingestion 

- CloudTrail logs
- S3 Access logs
- ELB logs
- VPC Flow logs
- Kinesis Data Stream

The service catalog will be deploying the solution as per the resources deployed in the member account and user requirements of the logs that are to be ingested into the Elastic Cloud. Firstly a CFT will deploy resources in the Log-Archive account and then another CFT will deploy the required resources in the member account. Please note that the deployment of the CFT should be initiated from the Master account and all the required resources will be deployed to the Log Archive account and the member account using the CloudFormation Stackset.

### To deploy this mechanism:

In the Log Archive account, the Service Catalog will be deploying the following resources using the CFT elastic-queue-bucket-forwarder.yaml.

- S3 buckets to collect the S3 access logs from the member accounts
- S3 buckets to collect the Elastic Load Balancer logs from the member accounts
- S3 bucket to collect the VPC Flow logs from the member accounts
- SQS Queue to notify Elastic Serverless Forwarder for VPC Flow Logs, Elastic Load Balancer logs, and S3 Access logs from the respective S3 buckets.
- Policies for the S3 buckets and the SQS Queues.
- KMS key to encrypt the SQS queue messages
- Elastic Serverless Forwarder to ingest the logs to Elastic Cloud.
- S3 bucket for storing the config.yaml file required for Elastic integration
- Bootstrap lambda function which will run a Python script to create config file required for elastic integration and upload the file to S3 bucket
- Policies for the S3 bucket (used for storing the config.yaml file) and Role and policy for bootstrap lambda.
 
In the Member account, the Service Catalog will be deploying the following resources using the CFT elastic-buckets-kinesis-member.yaml.

- S3 buckets to store the VPC flow logs and S3 access logs. These logs will be replicated to the buckets in the log archive account.
- Policies for the S3 buckets
- If the user wants to ingest Kinesis Data Streams to Elastic Cloud, then the following resources will be deployed. Please note that in this case the Kinesis Data Stream should be deployed in the member account.
  - S3 bucket for storing the cofig.yaml file. 
  - Bootstrap Lambda function which will run a Python script to create config.yaml file required for elastic integration and upload the file to S3 bucket. 
  - Elastic Serverless Forwarder to ingest the Kinesis to Elastic Cloud.
  - Policies for the S3 bucket (used for storing the config.yaml file) and Role and policy for bootstrap lambda.
