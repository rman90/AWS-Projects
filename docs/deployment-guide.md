# 🚀 Deployment Guide - Daily Agenda Infrastructure

This guide provides step-by-step instructions for deploying the Daily Agenda infrastructure on AWS using CloudFormation.

---

## 📋 Table of Contents

1. [Prerequisites](#prerequisites)
2. [Pre-Deployment Checklist](#pre-deployment-checklist)
3. [Deployment Option 1: Non-Nested Stack](#deployment-option-1-non-nested-stack)
4. [Deployment Option 2: Nested Stacks](#deployment-option-2-nested-stacks)
5. [Post-Deployment Verification](#post-deployment-verification)
6. [Accessing the Application](#accessing-the-application)
7. [Troubleshooting](#troubleshooting)
8. [Cleanup](#cleanup)

---

## Prerequisites

### Required Tools

- **AWS Account** with appropriate permissions
- **AWS CLI** installed and configured (version 2.x recommended)
- **Git** for cloning the repository
- **IAM Permissions** to create:
  - VPC and networking resources
  - EC2 instances
  - Security Groups
  - CloudFormation stacks
  - SSM Parameters
  - S3 buckets (for nested stacks)

### IAM Role Requirements

Ensure the IAM role `ec2ssm` exists in your account with the following permissions:
- `AmazonSSMManagedInstanceCore` (managed policy)
- `AmazonSSMReadOnlyAccess` (managed policy)

If the role doesn't exist, create it:

```bash
# Create IAM role for EC2 with SSM access
aws iam create-role \
  --role-name ec2ssm \
  --assume-role-policy-document '{
    "Version": "2012-10-17",
    "Statement": [{
      "Effect": "Allow",
      "Principal": {"Service": "ec2.amazonaws.com"},
      "Action": "sts:AssumeRole"
    }]
  }'

# Attach SSM policies
aws iam attach-role-policy \
  --role-name ec2ssm \
  --policy-arn arn:aws:iam::aws:policy/AmazonSSMManagedInstanceCore

aws iam attach-role-policy \
  --role-name ec2ssm \
  --policy-arn arn:aws:iam::aws:policy/AmazonSSMReadOnlyAccess

# Create instance profile
aws iam create-instance-profile --instance-profile-name ec2ssm
aws iam add-role-to-instance-profile \
  --instance-profile-name ec2ssm \
  --role-name ec2ssm
```

### AWS CLI Configuration

Verify your AWS CLI is configured:

```bash
# Check AWS CLI version
aws --version

# Verify credentials
aws sts get-caller-identity

# Set default region (optional)
export AWS_DEFAULT_REGION=us-east-1
```

---

## Pre-Deployment Checklist

- [ ] AWS CLI installed and configured
- [ ] IAM role `ec2ssm` exists with proper permissions
- [ ] Sufficient EC2 instance limits in your account (at least 1 t2.micro)
- [ ] VPC limit not exceeded (default: 5 per region)
- [ ] Repository cloned locally
- [ ] Decided on deployment approach (non-nested vs. nested)

---

## Deployment Option 1: Non-Nested Stack

### Step 1: Clone the Repository

```bash
git clone https://github.com/rman90/AWS-Projects.git
cd AWS-Projects
```

### Step 2: Review the Template

```bash
# View the template
cat non-nested-template/daily-agenda-stack.yaml

# Validate the template syntax
aws cloudformation validate-template \
  --template-body file://non-nested-template/daily-agenda-stack.yaml
```

### Step 3: Deploy the Stack

```bash
# Deploy with default parameters
aws cloudformation create-stack \
  --stack-name daily-agenda \
  --template-body file://non-nested-template/daily-agenda-stack.yaml \
  --parameters \
    ParameterKey=DailyAgendaParameter,ParameterValue="Team standup,Code review,Deploy to production" \
    ParameterKey=LabUserRoleName,ParameterValue=LabRole \
  --capabilities CAPABILITY_IAM \
  --tags Key=Project,Value=DailyAgenda Key=Environment,Value=Development

# Alternative: Deploy with custom agenda items
aws cloudformation create-stack \
  --stack-name daily-agenda \
  --template-body file://non-nested-template/daily-agenda-stack.yaml \
  --parameters \
    ParameterKey=DailyAgendaParameter,ParameterValue="Morning meeting,Client presentation,Team lunch,Afternoon coding session" \
    ParameterKey=LabUserRoleName,ParameterValue=LabRole \
  --capabilities CAPABILITY_IAM
```

### Step 4: Monitor Stack Creation

```bash
# Watch stack events in real-time
aws cloudformation describe-stack-events \
  --stack-name daily-agenda \
  --query 'StackEvents[*].[Timestamp,ResourceStatus,ResourceType,LogicalResourceId]' \
  --output table

# Check stack status
aws cloudformation describe-stacks \
  --stack-name daily-agenda \
  --query 'Stacks[0].StackStatus' \
  --output text

# Wait for stack creation to complete (blocking command)
aws cloudformation wait stack-create-complete \
  --stack-name daily-agenda
```

### Step 5: Retrieve Outputs

```bash
# Get all stack outputs
aws cloudformation describe-stacks \
  --stack-name daily-agenda \
  --query 'Stacks[0].Outputs' \
  --output table

# Get just the website URL
aws cloudformation describe-stacks \
  --stack-name daily-agenda \
  --query 'Stacks[0].Outputs[?OutputKey==`URL`].OutputValue' \
  --output text
```

---

## Deployment Option 2: Nested Stacks

### Step 1: Clone the Repository

```bash
git clone https://github.com/rman90/AWS-Projects.git
cd AWS-Projects
```

### Step 2: Create S3 Bucket for Templates

Nested stacks require templates to be stored in S3.

```bash
# Get your AWS account ID
export AWS_ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
export AWS_REGION=$(aws configure get region)

# Create S3 bucket (bucket name must be unique)
aws s3 mb s3://templates-${AWS_ACCOUNT_ID} --region ${AWS_REGION}

# Enable versioning (optional but recommended)
aws s3api put-bucket-versioning \
  --bucket templates-${AWS_ACCOUNT_ID} \
  --versioning-configuration Status=Enabled
```

### Step 3: Upload Nested Templates to S3

```bash
# Upload network stack template
aws s3 cp nested-stacks/network-stack.yaml \
  s3://templates-${AWS_ACCOUNT_ID}/network-stack.yaml

# Upload application stack template
aws s3 cp nested-stacks/application-stack.yaml \
  s3://templates-${AWS_ACCOUNT_ID}/application-stack.yaml

# Verify uploads
aws s3 ls s3://templates-${AWS_ACCOUNT_ID}/
```

### Step 4: Validate Templates

```bash
# Validate parent stack
aws cloudformation validate-template \
  --template-body file://nested-stacks/parent-stack.yaml

# Validate network stack
aws cloudformation validate-template \
  --template-body file://nested-stacks/network-stack.yaml

# Validate application stack
aws cloudformation validate-template \
  --template-body file://nested-stacks/application-stack.yaml
```

### Step 5: Deploy Parent Stack

```bash
# Deploy with default parameters
aws cloudformation create-stack \
  --stack-name daily-agenda-nested \
  --template-body file://nested-stacks/parent-stack.yaml \
  --parameters \
    ParameterKey=DailyAgendaParameterValue,ParameterValue="Finish Q4 deliverables,Week 5 code review,Add MFA to production accounts" \
  --capabilities CAPABILITY_IAM \
  --tags Key=Project,Value=DailyAgenda Key=Environment,Value=Development Key=Architecture,Value=Nested

# Alternative: Custom agenda items
aws cloudformation create-stack \
  --stack-name daily-agenda-nested \
  --template-body file://nested-stacks/parent-stack.yaml \
  --parameters \
    ParameterKey=DailyAgendaParameterValue,ParameterValue="Sprint planning,Architecture review,Security audit,Performance testing" \
  --capabilities CAPABILITY_IAM
```

### Step 6: Monitor Nested Stack Creation

```bash
# Monitor parent stack
aws cloudformation describe-stack-events \
  --stack-name daily-agenda-nested \
  --query 'StackEvents[*].[Timestamp,ResourceStatus,ResourceType,LogicalResourceId]' \
  --output table

# List all stacks (including nested)
aws cloudformation list-stacks \
  --stack-status-filter CREATE_IN_PROGRESS CREATE_COMPLETE \
  --query 'StackSummaries[*].[StackName,StackStatus]' \
  --output table

# Wait for completion
aws cloudformation wait stack-create-complete \
  --stack-name daily-agenda-nested
```

### Step 7: Retrieve Outputs

```bash
# Get parent stack outputs
aws cloudformation describe-stacks \
  --stack-name daily-agenda-nested \
  --query 'Stacks[0].Outputs' \
  --output table

# Get website URL
URL=$(aws cloudformation describe-stacks \
  --stack-name daily-agenda-nested \
  --query 'Stacks[0].Outputs[?OutputKey==`URL`].OutputValue' \
  --output text)

echo "Website URL: $URL"
```

---

## Post-Deployment Verification

### Verify Infrastructure Components

```bash
# Verify VPC
aws ec2 describe-vpcs \
  --filters "Name=tag:Name,Values=DailyAgenda-VPC" \
  --query 'Vpcs[*].[VpcId,CidrBlock,State]' \
  --output table

# Verify Subnet
aws ec2 describe-subnets \
  --filters "Name=tag:Name,Values=DailyAgenda-PublicSubnet" \
  --query 'Subnets[*].[SubnetId,CidrBlock,AvailabilityZone]' \
  --output table

# Verify Internet Gateway
aws ec2 describe-internet-gateways \
  --filters "Name=tag:Name,Values=DailyAgenda-IGW" \
  --query 'InternetGateways[*].[InternetGatewayId,Attachments[0].State]' \
  --output table

# Verify EC2 Instance
aws ec2 describe-instances \
  --filters "Name=tag:Name,Values=daily-agenda-service" \
  --query 'Reservations[*].Instances[*].[InstanceId,State.Name,PublicIpAddress,PublicDnsName]' \
  --output table

# Verify Security Group
aws ec2 describe-security-groups \
  --filters "Name=tag:Name,Values=DailyAgenda-WebServer-SG" \
  --query 'SecurityGroups[*].[GroupId,GroupName,VpcId]' \
  --output table

# Verify SSM Parameter
aws ssm get-parameter \
  --name /daily_agenda \
  --query 'Parameter.[Name,Type,Value]' \
  --output table
```

### Check EC2 Instance Status

```bash
# Get instance ID
INSTANCE_ID=$(aws ec2 describe-instances \
  --filters "Name=tag:Name,Values=daily-agenda-service" \
  --query 'Reservations[0].Instances[0].InstanceId' \
  --output text)

# Check instance status
aws ec2 describe-instance-status \
  --instance-ids $INSTANCE_ID \
  --query 'InstanceStatuses[0].[InstanceStatus.Status,SystemStatus.Status]' \
  --output table

# View instance console output (for troubleshooting)
aws ec2 get-console-output \
  --instance-id $INSTANCE_ID \
  --output text
```

---

## Accessing the Application

### Get the Website URL

```bash
# For non-nested stack
URL=$(aws cloudformation describe-stacks \
  --stack-name daily-agenda \
  --query 'Stacks[0].Outputs[?OutputKey==`URL`].OutputValue' \
  --output text)

# For nested stack
URL=$(aws cloudformation describe-stacks \
  --stack-name daily-agenda-nested \
  --query 'Stacks[0].Outputs[?OutputKey==`URL`].OutputValue' \
  --output text)

echo "Access your application at: $URL"
```

### Test HTTP Connectivity

```bash
# Test with curl
curl -I $URL

# Expected response: HTTP/1.1 200 OK

# View the full page
curl $URL
```

### Open in Browser

```bash
# macOS
open $URL

# Linux with xdg-open
xdg-open $URL

# Windows (WSL)
explorer.exe $URL
```

### Expected Output

You should see a web page with:
- A banner image
- "Daily Agenda" heading
- A bulleted list of your agenda items

---

## Troubleshooting

### Stack Creation Failed

```bash
# View stack events to identify the failure
aws cloudformation describe-stack-events \
  --stack-name daily-agenda \
  --query 'StackEvents[?ResourceStatus==`CREATE_FAILED`]' \
  --output table

# Get detailed error message
aws cloudformation describe-stack-events \
  --stack-name daily-agenda \
  --query 'StackEvents[?ResourceStatus==`CREATE_FAILED`].[LogicalResourceId,ResourceStatusReason]' \
  --output table
```

### EC2 Instance Not Accessible

```bash
# Check security group rules
aws ec2 describe-security-groups \
  --filters "Name=tag:Name,Values=DailyAgenda-WebServer-SG" \
  --query 'SecurityGroups[0].IpPermissions' \
  --output json

# Verify instance has public IP
aws ec2 describe-instances \
  --filters "Name=tag:Name,Values=daily-agenda-service" \
  --query 'Reservations[0].Instances[0].[PublicIpAddress,PublicDnsName]' \
  --output table

# Check if Apache is running (requires SSM Session Manager)
aws ssm send-command \
  --instance-ids $INSTANCE_ID \
  --document-name "AWS-RunShellScript" \
  --parameters 'commands=["systemctl status httpd"]'
```

### Website Shows Error

```bash
# Check CloudFormation Init logs on the instance
aws ssm start-session --target $INSTANCE_ID

# Once connected to the instance:
sudo cat /var/log/cfn-init.log
sudo cat /var/log/cfn-init-cmd.log
sudo systemctl status httpd
sudo cat /var/log/httpd/error_log
```

### Nested Stack Template Not Found

```bash
# Verify templates are in S3
aws s3 ls s3://templates-${AWS_ACCOUNT_ID}/

# Check S3 bucket permissions
aws s3api get-bucket-policy --bucket templates-${AWS_ACCOUNT_ID}

# Re-upload templates if needed
aws s3 cp nested-stacks/network-stack.yaml s3://templates-${AWS_ACCOUNT_ID}/ --force
aws s3 cp nested-stacks/application-stack.yaml s3://templates-${AWS_ACCOUNT_ID}/ --force
```

---

## Cleanup

### Delete Non-Nested Stack

```bash
# Delete the stack
aws cloudformation delete-stack --stack-name daily-agenda

# Wait for deletion to complete
aws cloudformation wait stack-delete-complete --stack-name daily-agenda

# Verify deletion
aws cloudformation describe-stacks --stack-name daily-agenda
# Expected: Stack not found error
```

### Delete Nested Stack

```bash
# Delete parent stack (automatically deletes nested stacks)
aws cloudformation delete-stack --stack-name daily-agenda-nested

# Wait for deletion
aws cloudformation wait stack-delete-complete --stack-name daily-agenda-nested

# Clean up S3 bucket
aws s3 rm s3://templates-${AWS_ACCOUNT_ID}/ --recursive
aws s3 rb s3://templates-${AWS_ACCOUNT_ID}

# Verify all stacks are deleted
aws cloudformation list-stacks \
  --stack-status-filter DELETE_COMPLETE \
  --query 'StackSummaries[?contains(StackName, `daily-agenda`)].[StackName,StackStatus]' \
  --output table
```

### Manual Cleanup (if needed)

If stack deletion fails, manually delete resources:

```bash
# Delete EC2 instance
aws ec2 terminate-instances --instance-ids $INSTANCE_ID

# Delete Security Group
SG_ID=$(aws ec2 describe-security-groups \
  --filters "Name=tag:Name,Values=DailyAgenda-WebServer-SG" \
  --query 'SecurityGroups[0].GroupId' \
  --output text)
aws ec2 delete-security-group --group-id $SG_ID

# Delete SSM Parameter
aws ssm delete-parameter --name /daily_agenda

# Delete VPC (after all resources are removed)
VPC_ID=$(aws ec2 describe-vpcs \
  --filters "Name=tag:Name,Values=DailyAgenda-VPC" \
  --query 'Vpcs[0].VpcId' \
  --output text)
aws ec2 delete-vpc --vpc-id $VPC_ID
```

---

## Additional Resources

- [AWS CloudFormation Documentation](https://docs.aws.amazon.com/cloudformation/)
- [AWS CLI Command Reference](https://docs.aws.amazon.com/cli/latest/reference/)
- [CloudFormation Best Practices](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/best-practices.html)
- [Nested Stacks Documentation](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/using-cfn-nested-stacks.html)

---

**Need Help?** Open an issue on [GitHub](https://github.com/rman90/AWS-Projects/issues)
