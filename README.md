# aws-security-quickstart



# Security Quick Start Guide - Foundational

## Overview

AWS provide services that help secure your workloads and applications in the cloud. The implementation guide discusses configuration steps for deploying the Security Quick Start solution in the Amazon Web Services (AWS) Cloud. The solution provides you with ready-to-deploy architecture for implementation of “Foundational” services  in the AWS Security Maturity Model (https://maturitymodel.security.aws.dev/en/model/). The solution uses AWS CloudFormation template to automatically launch and configure security services within single AWS account. 

## Architecture Overview

Deploying this solution with default parameters builds the following environment in your account.

Insert diagram here

1. *AWS Key Management Service (KMS).* To enable encryption-at-rest for common AWS services.
    1. Compliance check with AWS Config
    2. services covered: EBS, RDS, S3
2. *AWS Config.* To enable configuration monitoring with managed rules.
    1. services covered: EBS, RDS, S3
    2. scope: encryption and backup check
3. *Amazon Inspector.* To enable vulnerability assessment and network accessibility check for EC2 and ECR.
    1. enable using Lambda.
4. *AWS Backup.* To enable backup for common AWS services.
    1. services covered: EBS, RDS, S3
    2. plan included: daily-35days-retention, weekly-1yr-retention
5. *AWS Systems Manager*
    1. features covered: 
        1. Fleet Manager, using EventBridge to trigger Systems Manager Automation
6. *Infrastructure security and resiliency.*
    1. VPC with multi-AZs and network segmentation

## Automated deployment

*Note:* AWS CloudFormation is used to automate the deployment of Security Quick Start solution:CloudFormation template (https://github.com/henrykhho-hkh/aws-security-quickstart.git)
Use this template to launch the solution and all associated components. 

*Time to deploy:* Approximately 10 minutes

Before launching this solution, review the services provisioned, configurations and other considerations in the guide to figure out which services best suit your needs. Follow the step-by-step instructions in this section to configure and deploy the solution in your account. 

1. Sign in to the AWS Management Console and select the button to create stack with new resources in CloudFormation.
2. On the *Create stack* page, upload the CloudFormation template and choose *Next.*
3. On the *Specify stack details* page, assign a name to your solution stack and specify an email for notifications to be delivered to. You will need to verify the email address before they will receive notifications. 
4. Under *Parameters,* review the parameters for this solution template and modify them as necessary. This solution uses the following default values.

|Parameter |Default Value |Description |
|----------|--------------|------------|
|EnableAWSConfigEvaluation|TRUE|Enable AWS Config with Managed Rules to assess, audit, and evaluate configurations of your resources.|
|EnableAmazonInspector|TRUE|Enable Amazon Inspector for vulnerability assessment and network accessibility.|
|EnableAWSBackup|TRUE|Enable AWS Backup to backup your common AWS resources.|
|EnableDailyBackup|TRUE|Enable daily backup at 00:00 every night with 35 days retention period for your common AWS resources.|
|EnableWeeklyBackup|TRUE|Enable weekly backup at 00:00 every Sunday night with 1 year retention period for your common AWS resources.|
|EnableFleetManager|TRUE|Enable Fleet Manager to manage your EC2 nodes.|
|EnableNetworkSegmentation|FALSE|Create network segments in a new Amazon VPC.|
|EnvironmentName|AWS-SecurityQuickStart-VPC|An environment name that is prefixed to resource names|
|VpcCIDR|10.192.0.0/16|CIDR range of the VPC.|
|PublicSubnet1CIDR|10.192.10.0/24|CIDR range for the public subnet in the first Availability Zone.|
|PublicSubnet2CIDR|10.192.11.0/24|CIDR range for the public subnet in the second Availability Zone.|
|PrivateSubnet1CIDR|10.192.20.0/24|CIDR range for the private subnet in the first Availability Zone.|
|PrivateSubnet2CIDR|10.192.21.0/24|CIDR range for the private subnet in the second Availability Zone.|

1. Choose *Next.*
2. On the *Configure stack options* page, choose *Next.*
3. On the *Review* page, review and confirm the settings. Check the box acknowledging that the template will create AWS Identity and Access Management (IAM) resources. 
4. Choose *Create stack* to deploy the stack. 

You can view the status of the stack in the AWS CloudFormation Console in the *Status* column. You should receive a CREATE_COMPLETE status in approximately 10 minutes. 

## Post Deployment

### AWS Backup

AWS Backup protects your EC2, EBS, RDS and S3 resources based on the resource tags assigned. In order to include relevant resources in the backup plans, follow the below steps on AWS Console:

* In AWS Backup Console, opt-in AWS services for AWS Backup
* Assign tags (“backup enabled” : “yes”/“no”) to enable backup for resources
* Assign tags (“backup frequency” : “daily”/“weekly”) to define back frequencies

### AWS Fleet Manager

AWS Fleet Manager monitors and manages EC2 instances via IAM Instance Profile “DO-NOT-DELETE-SecurityQuickStartSSMRoleForInstances” and AWS Systems Manager Agents. For more information about IAM Instance Profile and AWS Systems Manager Agents, please refer to https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_use_switch-role-ec2_instance-profiles.html and https://docs.aws.amazon.com/systems-manager/latest/userguide/ssm-agent.html.

AWS System Manager Agents are pre-installed on some Amazon Machine Images (AMIs) provided by AWS. For the most updated list of AMIs, please refer to https://docs.aws.amazon.com/systems-manager/latest/userguide/ami-preinstalled-agent.html.  

For new EC2 instances, the IAM Instance Profile will be automatically applied with Amazon EventBridge and AWS Systems Manager Automation Document. However, for existing EC2 instances to be included in AWS Fleet Manager, follow the below steps on AWS Console or through CLI:

* Select the specific EC2 instance on AWS Console
* In “Actions”, select “Modify IAM role” under “Security
* Select “DO-NOT-DELETE-SecurityQuickStartSSMRoleForInstances” and click “Update IAM role”

If your EC2 instances have existing IAM Instance Profile, include the following AWS Managed IAM policies in your IAM Instance Profile:

* AmazonSSMManagedInstanceCore
* AmazonSSMPatchAssociation - to be confirmed if Patch Manager should be included

## Cost

You are responsible for the cost of the AWS services used to run this solution. The total cost for this service will depend on the number of services you enable and the size of your workload. The solution uses the following AWS components:

Service	Free Tier	Pricing
AWS Config	--	Refer to Pricing - AWS Config - Amazon Web Services (AWS) (https://aws.amazon.com/config/pricing/?did=ap_card&trk=ap_card) for pricing of AWS Config
AWS Fleet Manager	Free	Free
AWS EventBridge	--	Refer to Pricing - AWS EventBridge - Amazon Web Services (AWS) (https://aws.amazon.com/eventbridge/pricing/?did=ap_card&trk=ap_card) for pricing of Amazon EventBridge
AWS Lambda	1 million free requests per month	Refer to Pricing - AWS Lambda - Amazon Web Services (AWS) (https://aws.amazon.com/lambda/pricing/?did=ap_card&trk=ap_card) for pricing of AWS Lambda beyond the free tier
AWS Backup	--	Refer to Pricing - AWS Backup - Amazon Web Services (AWS) (https://aws.amazon.com/config/pricing/?did=ap_card&trk=ap_card) for pricing of AWS Backup
AWS VPC	--	Refer to Pricing - AWS VPC - Amazon Web Services (AWS) (https://aws.amazon.com/vpc/pricing/?did=ap_card&trk=ap_card) for pricing of AWS Backup

*Cost per month for a small to medium organization:* 
As of Oct 2022, the cost for running this solution, with default settings in the Asia Pacific (Singapore) Region is approximately *$1,002/month*. A detailed breakdown of this cost estimate is provided in the following tables.

Assumptions:

* Assume there is no restore request for AWS Backup

AWS service	Components	Unit	Quantity	Unit Price	Monthly Price
AWS Config	Configuration items	Number of items	10,000	$0.003	$30
Rules evaluations	Number of evaluations	50,000	$0.001	$50
AWS Backup	EBS volume snapshot	GB-month	3,000	$0.050	$150
	RDS database snapshot	GB-month	5,000	$0.095	$475
	S3 backup	GB-month	2,000	$0.060	$120
AWS VPC	NAT Gateway	Number of gateways	2	$0.059	$0.12
	Data processed	GB-month	3,000	$0.059	$177
Total					$1,002

*Note: Other AWS services include AWS Lambda and Amazon EventBridge is priced at the free tier

Uninstall the solution

You can uninstall the solution from the AWS Management Console or by using the AWS Command Line Interface. You must manually delete the S3 buckets and CloudFormation stacks created by this solution. 

Using the AWS Management Console

1. Sign in to the AWS CloudFormation console.
2. On the *Stacks* page, select the AWS security quick start stack and choose delete 

Using the AWS Command Line Interface

Determine whether the AWS Command Line Interface (AWS CLI) is available in your environment. For installation instructions, refer to What Is the AWS Command Line Interface (https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-welcome.html) in the AWS CLI User Guide. After confirming that the AWS CLI is available, run the following commands. Replace your-stack-name with the name of your CloudFormation stack. 
 
$ aws cloudformation delete-stack —stack-name your-stack-name