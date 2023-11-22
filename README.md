# Elastic-Spektra


## Deployment

The solution can be deployed to customer Organizational Units (OUs) so it will collect logs from all the child accounts. We have considered the following inputs for log ingestion 

- CloudTrail - Will be created by the Control Tower deployed in the Master Account
- S3 Access logs - Deployed in the Member account
- Elastic Load Balancer - Deployed in the Member account
- VPC Flow logs - Deployed in the Member account
- Kinesis Data Stream - Deployed in the Member account

The service catalog will be deploying the solution as per the resources deployed in the member account and user requirements of the logs that are to be ingested into the Elastic Cloud. Firstly the CFT will deploy resources in the Log-Archive account. Please note that the deployment of the CFT should be initiated from the Master account, all the required resources will be deployed to the Log Archive account and the member account will be created by using the CloudFormation Stackset.

### To deploy this mechanism:

In the Log-Archive account, the Service catalog will deploy S3 buckets that will collect the logs from the required input resources and SQS queues to ingest the logs from S3 buckets into Elastic Cloud. Following AWS CloudFormation Templates will be deployed in the Log Archive account.

- elastic-queues-and-buckets: This template deploys the following resources.
  - 2 S3 buckets to collect the S3 access logs and the Elastic Load Balancer logs from the member accounts
  - 2 SQS Queues for ingesting logs from S3 buckets into Elastic Cloud. This will ingest S3 access logs and the ELB logs from the respective buckets
  - Policies for the S3 buckets and the SQS Queues
  - KMS key to encrypt the SQS queues messages
 
- elastic-vpcflow-buckets-queues: This template deploys the following resources.
  - S3 bucket to collect the VPC Flow logs from the member accounts
  - SQS Queue for ingesting logs from S3 buckets into Elastic Cloud. This will ingest the VPC Flow logs from the respective bucket
  - Policies for the S3 bucket and the SQS Queue
  - Elastic Serverless Forwarder
  - S3 bucket for storing the config.yaml file required for Elastic integration
  - Bootstrap lambda which will deploy a Python script to create the config file and upload it to S3 bucket
  - Policies for the S3 bucket (used for storing the config.yaml file) and Role and policy for bootstrap lambda.
