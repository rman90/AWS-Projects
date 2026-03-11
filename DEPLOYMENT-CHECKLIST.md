# ✅ Deployment Checklist

Use this checklist to ensure a smooth deployment of the Daily Agenda infrastructure.

---

## Pre-Deployment Checklist

### AWS Account Setup

- [ ] AWS account created and active
- [ ] AWS CLI installed (version 2.x recommended)
- [ ] AWS CLI configured with credentials
  ```bash
  aws configure
  ```
- [ ] Verified AWS credentials work
  ```bash
  aws sts get-caller-identity
  ```
- [ ] Set default region (optional)
  ```bash
  export AWS_DEFAULT_REGION=us-east-1
  ```

### IAM Permissions

- [ ] User/role has CloudFormation permissions
- [ ] User/role has EC2 permissions
- [ ] User/role has VPC permissions
- [ ] User/role has IAM permissions (for creating instance profiles)
- [ ] User/role has S3 permissions (for nested stacks)

### IAM Role for EC2

- [ ] IAM role `ec2ssm` exists
- [ ] Role has `AmazonSSMManagedInstanceCore` policy attached
- [ ] Role has `AmazonSSMReadOnlyAccess` policy attached
- [ ] Instance profile `ec2ssm` created
- [ ] Role added to instance profile

**Quick Setup:**
```bash
aws iam create-role --role-name ec2ssm \
  --assume-role-policy-document '{
    "Version": "2012-10-17",
    "Statement": [{
      "Effect": "Allow",
      "Principal": {"Service": "ec2.amazonaws.com"},
      "Action": "sts:AssumeRole"
    }]
  }'

aws iam attach-role-policy --role-name ec2ssm \
  --policy-arn arn:aws:iam::aws:policy/AmazonSSMManagedInstanceCore

aws iam attach-role-policy --role-name ec2ssm \
  --policy-arn arn:aws:iam::aws:policy/AmazonSSMReadOnlyAccess

aws iam create-instance-profile --instance-profile-name ec2ssm
aws iam add-role-to-instance-profile --instance-profile-name ec2ssm --role-name ec2ssm
```

### Service Limits

- [ ] EC2 instance limit allows at least 1 t2.micro
- [ ] VPC limit not exceeded (default: 5 per region)
- [ ] EBS volume limit allows 2 volumes
- [ ] Elastic IP limit allows 1 (if needed)

### Repository Setup

- [ ] Repository cloned locally
  ```bash
  git clone https://github.com/rman90/AWS-Projects.git
  cd AWS-Projects
  ```
- [ ] Templates validated
  ```bash
  aws cloudformation validate-template \
    --template-body file://non-nested-template/daily-agenda-stack.yaml
  ```

---

## Deployment Checklist - Non-Nested Stack

### Step 1: Prepare Parameters

- [ ] Decided on stack name (e.g., `daily-agenda`)
- [ ] Prepared agenda items (comma-separated)
- [ ] Confirmed LabUserRoleName (default: `LabRole`)

### Step 2: Deploy Stack

- [ ] Executed create-stack command
  ```bash
  aws cloudformation create-stack \
    --stack-name daily-agenda \
    --template-body file://non-nested-template/daily-agenda-stack.yaml \
    --parameters \
      ParameterKey=DailyAgendaParameter,ParameterValue="Team standup,Code review,Deploy" \
      ParameterKey=LabUserRoleName,ParameterValue=LabRole \
    --capabilities CAPABILITY_IAM
  ```
- [ ] Stack creation initiated successfully
- [ ] Stack ID received

### Step 3: Monitor Deployment

- [ ] Monitoring stack events
  ```bash
  aws cloudformation describe-stack-events --stack-name daily-agenda
  ```
- [ ] Checked for CREATE_FAILED events
- [ ] Waited for CREATE_COMPLETE status
  ```bash
  aws cloudformation wait stack-create-complete --stack-name daily-agenda
  ```

### Step 4: Verify Deployment

- [ ] Stack status is CREATE_COMPLETE
- [ ] All resources created successfully
- [ ] Retrieved stack outputs
  ```bash
  aws cloudformation describe-stacks --stack-name daily-agenda \
    --query 'Stacks[0].Outputs' --output table
  ```
- [ ] Website URL obtained

### Step 5: Test Application

- [ ] Accessed website URL in browser
- [ ] Website loads successfully
- [ ] Banner image displays
- [ ] Agenda items display correctly
- [ ] No errors in browser console

---

## Deployment Checklist - Nested Stacks

### Step 1: Prepare S3 Bucket

- [ ] Got AWS account ID
  ```bash
  export AWS_ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
  ```
- [ ] Created S3 bucket
  ```bash
  aws s3 mb s3://templates-${AWS_ACCOUNT_ID}
  ```
- [ ] Enabled versioning (optional)
  ```bash
  aws s3api put-bucket-versioning \
    --bucket templates-${AWS_ACCOUNT_ID} \
    --versioning-configuration Status=Enabled
  ```

### Step 2: Upload Templates

- [ ] Uploaded network stack
  ```bash
  aws s3 cp nested-stacks/network-stack.yaml s3://templates-${AWS_ACCOUNT_ID}/
  ```
- [ ] Uploaded application stack
  ```bash
  aws s3 cp nested-stacks/application-stack.yaml s3://templates-${AWS_ACCOUNT_ID}/
  ```
- [ ] Verified uploads
  ```bash
  aws s3 ls s3://templates-${AWS_ACCOUNT_ID}/
  ```

### Step 3: Validate Templates

- [ ] Validated parent stack
  ```bash
  aws cloudformation validate-template \
    --template-body file://nested-stacks/parent-stack.yaml
  ```
- [ ] Validated network stack
  ```bash
  aws cloudformation validate-template \
    --template-body file://nested-stacks/network-stack.yaml
  ```
- [ ] Validated application stack
  ```bash
  aws cloudformation validate-template \
    --template-body file://nested-stacks/application-stack.yaml
  ```

### Step 4: Deploy Parent Stack

- [ ] Prepared agenda items parameter
- [ ] Executed create-stack command
  ```bash
  aws cloudformation create-stack \
    --stack-name daily-agenda-nested \
    --template-body file://nested-stacks/parent-stack.yaml \
    --parameters \
      ParameterKey=DailyAgendaParameterValue,ParameterValue="Q4 deliverables,Code review,MFA" \
    --capabilities CAPABILITY_IAM
  ```
- [ ] Stack creation initiated

### Step 5: Monitor Nested Stacks

- [ ] Monitored parent stack
- [ ] Monitored network nested stack
- [ ] Monitored application nested stack
- [ ] All stacks reached CREATE_COMPLETE
  ```bash
  aws cloudformation wait stack-create-complete --stack-name daily-agenda-nested
  ```

### Step 6: Verify Deployment

- [ ] Parent stack status is CREATE_COMPLETE
- [ ] Network stack status is CREATE_COMPLETE
- [ ] Application stack status is CREATE_COMPLETE
- [ ] Retrieved outputs from parent stack
- [ ] Website URL obtained

### Step 7: Test Application

- [ ] Accessed website URL
- [ ] Application works correctly
- [ ] Agenda items display
- [ ] No errors

---

## Post-Deployment Verification

### Infrastructure Verification

- [ ] VPC created
  ```bash
  aws ec2 describe-vpcs --filters "Name=tag:Name,Values=DailyAgenda-VPC"
  ```
- [ ] Subnet created
  ```bash
  aws ec2 describe-subnets --filters "Name=tag:Name,Values=DailyAgenda-PublicSubnet"
  ```
- [ ] Internet Gateway attached
  ```bash
  aws ec2 describe-internet-gateways --filters "Name=tag:Name,Values=DailyAgenda-IGW"
  ```
- [ ] EC2 instance running
  ```bash
  aws ec2 describe-instances --filters "Name=tag:Name,Values=daily-agenda-service"
  ```
- [ ] Security group configured
  ```bash
  aws ec2 describe-security-groups --filters "Name=tag:Name,Values=DailyAgenda-WebServer-SG"
  ```
- [ ] SSM parameter exists
  ```bash
  aws ssm get-parameter --name /daily_agenda
  ```

### Application Verification

- [ ] HTTP connectivity works
  ```bash
  curl -I http://your-ec2-url
  ```
- [ ] Website returns 200 OK
- [ ] HTML content loads
- [ ] PHP script executes
- [ ] SSM parameter retrieved
- [ ] Agenda items rendered

### Security Verification

- [ ] Security group allows HTTP (80)
- [ ] Security group restricts egress
- [ ] No SSH access configured (good!)
- [ ] IAM role attached to instance
- [ ] No hardcoded credentials in code

---

## Cleanup Checklist

### Non-Nested Stack Cleanup

- [ ] Deleted stack
  ```bash
  aws cloudformation delete-stack --stack-name daily-agenda
  ```
- [ ] Waited for deletion
  ```bash
  aws cloudformation wait stack-delete-complete --stack-name daily-agenda
  ```
- [ ] Verified stack deleted
- [ ] Verified all resources removed

### Nested Stack Cleanup

- [ ] Deleted parent stack
  ```bash
  aws cloudformation delete-stack --stack-name daily-agenda-nested
  ```
- [ ] Waited for deletion
  ```bash
  aws cloudformation wait stack-delete-complete --stack-name daily-agenda-nested
  ```
- [ ] Verified nested stacks deleted
- [ ] Cleaned up S3 bucket
  ```bash
  aws s3 rm s3://templates-${AWS_ACCOUNT_ID}/ --recursive
  aws s3 rb s3://templates-${AWS_ACCOUNT_ID}
  ```
- [ ] Verified all resources removed

---

## Troubleshooting Checklist

### Stack Creation Failed

- [ ] Checked stack events for errors
  ```bash
  aws cloudformation describe-stack-events --stack-name daily-agenda \
    --query 'StackEvents[?ResourceStatus==`CREATE_FAILED`]'
  ```
- [ ] Reviewed error messages
- [ ] Verified IAM permissions
- [ ] Checked service limits
- [ ] Validated template syntax

### Instance Not Accessible

- [ ] Verified instance is running
- [ ] Checked security group rules
- [ ] Verified public IP assigned
- [ ] Tested with curl
- [ ] Checked instance logs
  ```bash
  aws ec2 get-console-output --instance-id i-xxxxx
  ```

### Application Not Working

- [ ] Checked Apache status
- [ ] Reviewed cfn-init logs
- [ ] Verified SSM parameter exists
- [ ] Checked IAM role permissions
- [ ] Reviewed application logs

---

## Success Criteria

✅ **Deployment Successful When:**

- [ ] CloudFormation stack status is CREATE_COMPLETE
- [ ] All resources created without errors
- [ ] EC2 instance is running
- [ ] Website is accessible via HTTP
- [ ] Agenda items display correctly
- [ ] No errors in logs
- [ ] All verification tests pass

---

## Documentation Checklist

- [ ] README.md reviewed
- [ ] Deployment guide consulted
- [ ] Architecture explanation understood
- [ ] Quick start guide followed
- [ ] Troubleshooting guide referenced

---

## Next Steps After Deployment

- [ ] Bookmark website URL
- [ ] Document any customizations
- [ ] Plan for enhancements
- [ ] Consider high availability improvements
- [ ] Evaluate cost optimization opportunities
- [ ] Schedule cleanup (if temporary)

---

**Deployment Date:** _______________  
**Stack Name:** _______________  
**Region:** _______________  
**Website URL:** _______________

---

**Need Help?** Refer to the [Deployment Guide](docs/deployment-guide.md) or open an issue on GitHub.
