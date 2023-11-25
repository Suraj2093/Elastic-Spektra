# Elastic-Spektra

## Pre-requites

- You need to setup the Control Tower in the AWS account. This will setup the Log Archive Account and deploy the CloudTrail.
- Before the Service catalog is deployed, please have all the required resources like Elastic Load Balancer, Kinesis Data Stream, S3 buckets, etc.
- Please make sure the deployment region of the Service Catalog is us-east-1 because we are using a zip file that contains the Python script and required packages to create the config.yaml file and currently, this zip file is in a public bucket in the us-east-1 region, deploying the CFT in any other region will fail.
- You must have an Elastic Cloud ID to ingest the logs to the Elastic Cloud.

## Deployment

The service catalog can be deployed to the Log Archive account and the customer member accounts which are part of the AWS Organization so it will collect logs accordingly. We have considered the following inputs for log ingestion 

- CloudTrail logs
- S3 Access logs
- ELB logs
- VPC Flow logs
- Kinesis Data Stream
- CloudWatch logs

The service catalog will be deploying the solution as per the resources deployed in the member account and user requirements of the logs that are to be ingested into the Elastic Cloud. The CFT will deploy resources in the Log-Archive account and then another CFT will deploy the required resources in the member account. Please note that the deployment of the CFT should be initiated from the Master account and all the required resources will be deployed to the Log Archive account and the member account using the CloudFormation Stackset.

### Deployment Steps:

The src folder has 2 CFT files that need to be downloaded. Following are the instructions to deploy the Service Catalog solution in your AWS account.

1. Login to the AWS account and go to Service Catalog
2. Create portfolio to organize your products and distribute them to end users. Add products to a portfolio and grant permissions to allow users to view and launch products. Please refer to the [Create a Portfolio](https://docs.aws.amazon.com/servicecatalog/latest/adminguide/getstarted-portfolio.html).
3. Download the CFTs elastic-ingestion-log-archive.yaml and  elastic-ingestion-member.yaml from the src folder.
4. Once the portfolio is created, create 2 products - 1 for deploying resources in the Log Archive account and another one for deploying the resources in the member account. Please refer to the [Create a new product in the portfolio](https://docs.aws.amazon.com/servicecatalog/latest/adminguide/getstarted-product.html). Upload the CFT downloaded from the src folder while creating the products.
5. Create constraints for both the products created. For the product created for elastic-ingestion-log-archive.yaml, select **StackSet** as Constraint Type and enter the Log Archive account ID, so that the product can just be deployed in that particular account only. For the product created for elastic-ingestion-member.yaml, select **StackSet** as Constraint Type, and enter the member account IDs, so that the product can just be deployed only in the mentioned member accounts. Also, provide the region specification as **US EAST** for both the product constraints so the deployment can be only done in this region since the zip file that contains the Python script and required packages to create the config.yaml file is stored in a public bucket in us-east-1 region. Please refer to [Adding Constraints](https://docs.aws.amazon.com/servicecatalog/latest/adminguide/portfoliomgmt-constraints.html) for more details.
6. Now we are ready to deploy the products. To deploy the product to the Log Archive account, provide the following parameters:

    6.1 

7. To deploy the product to the member account, provide the following parameters:

    7.1 
   
9.  Once the deployment of the CFTs is completed, log into the Elastic dashboard to view the logs ingested.


### Technical details:

In the Log Archive account, the Service Catalog will be deploying the following resources using the CFT elastic-ingestion-log-archive.yaml.

- S3 buckets to collect the S3 access logs from the member accounts
- S3 buckets to collect the Elastic Load Balancer logs from the member accounts
- S3 bucket to collect the VPC Flow logs from the member accounts
- SQS Queue to notify Elastic Serverless Forwarder for VPC Flow Logs, Elastic Load Balancer logs, CloudTrail logs, and S3 Access logs from the respective S3 buckets.
- Policies for the S3 buckets and the SQS Queues.
- KMS key to encrypt the SQS queue messages
- Elastic Serverless Forwarder to ingest the logs to Elastic Cloud.
- S3 bucket for storing the config.yaml file required for Elastic integration
- Bootstrap lambda function which will run a Python script to create config file required for elastic integration and upload the file to S3 bucket
- Policies for the S3 bucket (used for storing the config.yaml file) and Role and policy for bootstrap lambda.
 
In the Member account, the Service Catalog will be deploying the following resources using the CFT elastic-buckets-kinesis-member.yaml.

- S3 buckets to store the VPC flow logs and S3 access logs. These logs will be replicated to the buckets in the log archive account.
- Policies for the S3 buckets
- If the user wants to either ingest Kinesis Data Streams or Amazon Cloudwatch logs to Elastic Cloud, then the following resources will be deployed. Please note that in this case the Kinesis Data Stream/Amazon Cloudwatch should be deployed in the member account as per the requirement.
  - S3 bucket for storing the cofig.yaml file. 
  - Bootstrap Lambda function which will run a Python script to create config.yaml file required for elastic integration and upload the file to S3 bucket. 
  - Elastic Serverless Forwarder to ingest the Kinesis Data Stream/Amazon Cloudwatch logs to Elastic Cloud as per requirement.
  - Policies for the S3 bucket (used for storing the config.yaml file) and Role and policy for bootstrap lambda.
 
