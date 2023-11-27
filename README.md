# Elastic-Spektra

## Architecture

The following architecture diagram outlines high-level components involved in ingesting logs from various AWS Services to Elastic Cloud.

1. Master Account: The AWS Master account will be primarily used as a deployment entry point for solution, AWS Control Tower management and to host Elastic forwarder for AWS Directory services-related logs, if enabled.
2. Log Archive Account: This account will be primarily used to store the logs in various S3 buckets through control tower integration, it will also host the primary Elastic Serverless forwarder and SQS to notify the logs from S3 buckets into Elastic Cloud.
3. Member Account(s): One or more member accounts may contain AWS resources, sending logs to the log-archive account. Member accounts may also host the Elastic serverless forwarder for Kinesis data stream/cloud watch logs ingestion to Elastic Cloud if required.

![alt text](images/elastic-arch.png)

## Pre-requites

- You need to set up the Control Tower in the AWS master account. This will set up the Log Archive account and deploy the CloudTrail.
- Before the Service catalog is deployed, please have all the required resources like Elastic Load Balancer, Kinesis Data Stream, S3 buckets, etc deployed.
- The forwarder Lambda function that creates the config.yaml file for Elastic Serverless Forwarder is written in Python, and requires a set of node modules. These required scripts and packages are zipped and uploaded in the src folder. Download the zip file **ElasticBootstrapLambdaLogArchiveAccount** from the folder log-archive-account in src and the zip file **ElasticBootstrapLambdaMemberAccount** from the folder member-account in src. Create S3 bucket in the region where you want to deploy the service catalog and upload these zip files in the bucket. The zip files need to have the following bucket policy so they can be shared with the organization and accessed by the forwarder Lambda function deployment. Please refer to the sample code below and provide the arn of the S3 bucket and the organization ID as required:

  ```
  {
     "Version": "2012-10-17",
     "Statement": [
     {
         "Sid": "AllowGetObject",
         "Effect": "Allow",
         "Principal": {
             "AWS": "*"
         },
         "Action": "s3:GetObject",
         "Resource": "arn:aws:s3::: s3-bucket-name/*",
         "Condition": {
             "StringEquals": {
                 "aws:PrincipalOrgID": "org-id "
             }
         }
      }
     ]
  }
  ```
  
- If you want to deploy Elastic Serverless forwarder in Amazon VPC, then you need to create the required VPC, subnets, and route tables in the account you are deploying the Service Catalog product.
- You must have an Elastic Cluster, Elastic Cloud ID, and Elastic API key to ingest the logs to the Elastic Cloud.

## Deployment

The service catalog must be deployed to the Log Archive account and the customer member accounts which are part of the AWS Organization so it can collect logs accordingly. We have considered the following inputs for log ingestion 

- CloudTrail logs
- S3 Access logs
- ELB logs
- VPC Flow logs
- Kinesis Data Stream
- CloudWatch logs

The service catalog will be deploying the solution as per the resources deployed in the member account and user requirements of the logs that are to be ingested into the Elastic Cloud. The CFT will deploy resources in the Log-Archive account and then another CFT will deploy the required resources in the member account. Please note that the deployment of the CFT should be initiated from the Master account and all the required resources will be deployed to the Log Archive account and the member account using the CloudFormation Stackset.

### Deployment Steps:

The src folder has 2 folders log-archive-account and member-account in which we have uploaded the required CFT files. Following are the instructions to deploy the Service Catalog solution in your AWS account.

1. Login to the AWS Master account.
2. Go to Service catalog and create a portfolio to organize your products and distribute them to end users. Please refer to the [Create a Portfolio](https://docs.aws.amazon.com/servicecatalog/latest/adminguide/getstarted-portfolio.html).
3. Download the CFTs **elastic-ingestion-log-archive.yaml** in log-archive-account folder from the src folder and **elastic-ingestion-member.yaml** in member-account folder from the src folder.
4. Create 2 products - 1 for deploying resources in the Log Archive account and another one for deploying the resources in the member account. Select the product type as **CloudFormation** and version source as **use a template file** and  upload the CFT downloaded from the src folder. Please refer to the [Create a new product in the portfolio](https://docs.aws.amazon.com/servicecatalog/latest/adminguide/getstarted-product.html).
5. Create constraints for both the products created. For the product created for elastic-ingestion-log-archive.yaml, select **StackSet** as Constraint Type and enter the Log Archive account ID, so that the product can just be deployed in that particular account only. For the product created for elastic-ingestion-member.yaml, select **StackSet** as Constraint Type, and enter the member account ID/IDs, so that the product can just be deployed only in the mentioned member accounts. Also, provide the region specification for both the product constraints so the deployment can be only done in the region where the bucket that contains the zip file was deployed earlier. Please refer to [Adding Constraints](https://docs.aws.amazon.com/servicecatalog/latest/adminguide/portfoliomgmt-constraints.html) for more details.
6. Once the products are added to the portfolio and constraints are added, you need to grant permission to allow yourself to view and launch products. Please refer to [Grant end users access to the portfolio](https://docs.aws.amazon.com/servicecatalog/latest/adminguide/getstarted-deploy.html) for more details.
7. Now we are ready to deploy the products. To deploy the products to the Log Archive account, go to the **products** page, you will see the products for which you have granted access. Select the product and click on **Launch product**, provide the following parameters, and click on **Launch product** again:

   7.1 Provisioned product name: Enter a unique name or select Generate name to provide a name automatically
   
   7.2 Select the product version
   
   7.3 Select the Log Archive account
   
   7.4 Select the region of deployment
   
   7.5 CloudTrail Bucket Name: Name of the AWS Control Tower created CloudTrail S3 bucket present in Log Archive Account

   7.6 CloudTrail Bucket ARN: ARN of the AWS Control Tower created CloudTrail S3 bucket present in Log Archive Account

   7.7 Elastic Cloud ID: Cloud ID of Elastic Cluster Deployment

   7.8 Elastic API Key: RESTful API to provide access to deployment CRUD actions

   7.9 Bootstrap Lambda Config Bucket: The S3 bucket name where the Bootstrap Lambda configuration zip file is stored.

   7.10 Bootstrap Lambda Config File Path: The path in the S3 bucket where the Lambda configuration zip file is located.

   7.11 AWS Organization ID: The ID of your AWS Organization.

   7.12 Deploy Serverless Forwarder In VPC: Select 'yes' if you want to deploy the Elastic Serverless Forwarder in VPC. This is an optional parameter.

   7.13 VPC ID: Enter the VPC ID in which you want to deploy the Elastic Serverless Forwarder. Provide value only if you have selected 'yes' for the parameter ***Deploy Serverless Forwarder In VPC***.

   7.14 Subnet IDs: Enter the Subnet IDs (comma-separated) for the Elastic Serverless Forwarder. Provide value only if you have selected 'yes' for the parameter ***Deploy Serverless Forwarder In VPC***.


8. To deploy the products to the Log Archive account, go to the **products** page, select the product and click on **Launch product**, provide the following parameters, and click on **Launch product** again::

   8.1 Provisioned product name: Enter a unique name or select Generate name to provide a name automatically
   
   8.2 Select the product version
   
   8.3 Select the member account where you to deploy the product
   
   8.4 Select the region of deployment

   8.5 Account ID: AWS Account Id of the log-archive account

   8.6 Logging Region: What region are the log buckets in? Region of the central logging bucket created in the Log archive account.

   8.7 AWS Organization ID: The ID of your AWS Organization

   8.8 Elastic Cloud ID: Cloud ID of Elastic Cluster Deployment

   8.9 Elastic API Key: RESTful API to provide access to deployment CRUD actions

   8.10 Bootstrap Lambda Config Bucket: The S3 bucket name where the Elastic Bootstrap Lambda configuration zip file is stored.

   8.11 Bootstrap Lambda Config File Path: The path in the S3 bucket where the Elastic Bootstrap Lambda configuration zip file is located.

   8.12 Ingest Kinesis Logs: Select 'yes' if you want to ingest Kinesis data streams. This is an optional parameter.

   8.13 Kinesis Data Stream ARNs: Comma-delimited list of Kinesis Data Stream ARNs. Provide value only if you have selected 'yes' for the parameter ***Ingest Kinesis Log***.

   8.14 Ingest CloudWatch Logs: Select 'yes' if you want to ingest CloudWatch Log Group logs. This is an optional parameter.

   8.15 CloudWatch LogGroup ARNs: Comma-delimited list of CloudWatch Log Group ARNs. Provide value only if you have selected 'yes' for the parameter ***Ingest CloudWatch Logs***.

   8.16 Deploy Serverless Forwarder In VPC: Select 'yes' if you want to deploy the Elastic Serverless Forwarder in VPC. This is an optional parameter.

   8.17 VPC ID: Enter the VPC ID in which you want to deploy the Elastic Serverless Forwarder. Provide value only if you have selected 'yes' for the parameter ***Deploy Serverless Forwarder In VPC***.

   8.18 Subnet IDs: Enter the Subnet IDs (comma-separated) for the Elastic Serverless Forwarder. Provide value only if you have selected 'yes' for the parameter ***Deploy Serverless Forwarder In VPC***.
   
15.  Once the deployment of the CFTs are completed, log into the Elastic dashboard to view the logs ingested.

### Technical details:

In the Log Archive account, the Service Catalog product will be deploying the following resources using the CFT elastic-ingestion-log-archive.yaml.

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

