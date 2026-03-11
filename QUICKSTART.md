# ⚡ Quick Start Guide

Get the Daily Agenda infrastructure running in under 5 minutes!

---

## 🚀 Fast Track Deployment

### Option 1: Non-Nested Stack (Simplest)

```bash
# Clone repository
git clone https://github.com/rman90/AWS-Projects.git
cd AWS-Projects

# Deploy stack
aws cloudformation create-stack \
  --stack-name daily-agenda \
  --template-body file://non-nested-template/daily-agenda-stack.yaml \
  --parameters \
    ParameterKey=DailyAgendaParameter,ParameterValue="Team standup,Code review,Deploy to production" \
    ParameterKey=LabUserRoleName,ParameterValue=LabRole \
  --capabilities CAPABILITY_IAM

# Wait for completion (5-10 minutes)
aws cloudformation wait stack-create-complete --stack-name daily-agenda

# Get website URL
aws cloudformation describe-stacks \
  --stack-name daily-agenda \
  --query 'Stacks[0].Outputs[?OutputKey==`URL`].OutputValue' \
  --output text
```

### Option 2: Nested Stacks (Production-Style)

```bash
# Clone repository
git clone https://github.com/rman90/AWS-Projects.git
cd AWS-Projects

# Setup
export AWS_ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
aws s3 mb s3://templates-${AWS_ACCOUNT_ID}

# Upload templates
aws s3 cp nested-stacks/network-stack.yaml s3://templates-${AWS_ACCOUNT_ID}/
aws s3 cp nested-stacks/application-stack.yaml s3://templates-${AWS_ACCOUNT_ID}/

# Deploy
aws cloudformation create-stack \
  --stack-name daily-agenda-nested \
  --template-body file://nested-stacks/parent-stack.yaml \
  --parameters \
    ParameterKey=DailyAgendaParameterValue,ParameterValue="Finish Q4 deliverables,Week 5 code review,Add MFA" \
  --capabilities CAPABILITY_IAM

# Wait for completion
aws cloudformation wait stack-create-complete --stack-name daily-agenda-nested

# Get URL
aws cloudformation describe-stacks \
  --stack-name daily-agenda-nested \
  --query 'Stacks[0].Outputs[?OutputKey==`URL`].OutputValue' \
  --output text
```

---

## 🧹 Quick Cleanup

### Delete Non-Nested Stack
```bash
aws cloudformation delete-stack --stack-name daily-agenda
aws cloudformation wait stack-delete-complete --stack-name daily-agenda
```

### Delete Nested Stack
```bash
aws cloudformation delete-stack --stack-name daily-agenda-nested
aws cloudformation wait stack-delete-complete --stack-name daily-agenda-nested
aws s3 rm s3://templates-${AWS_ACCOUNT_ID}/ --recursive
aws s3 rb s3://templates-${AWS_ACCOUNT_ID}
```

---

## 📊 Quick Verification

```bash
# Check stack status
aws cloudformation describe-stacks \
  --stack-name daily-agenda \
  --query 'Stacks[0].StackStatus'

# List all resources
aws cloudformation list-stack-resources \
  --stack-name daily-agenda \
  --query 'StackResourceSummaries[*].[ResourceType,LogicalResourceId,ResourceStatus]' \
  --output table

# Test website
URL=$(aws cloudformation describe-stacks \
  --stack-name daily-agenda \
  --query 'Stacks[0].Outputs[?OutputKey==`URL`].OutputValue' \
  --output text)
curl -I $URL
```

---

## 🔧 Common Issues

### Issue: IAM role 'ec2ssm' not found

**Solution:**
```bash
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

aws iam attach-role-policy \
  --role-name ec2ssm \
  --policy-arn arn:aws:iam::aws:policy/AmazonSSMManagedInstanceCore

aws iam create-instance-profile --instance-profile-name ec2ssm
aws iam add-role-to-instance-profile --instance-profile-name ec2ssm --role-name ec2ssm
```

### Issue: Stack creation failed

**Solution:**
```bash
# View error details
aws cloudformation describe-stack-events \
  --stack-name daily-agenda \
  --query 'StackEvents[?ResourceStatus==`CREATE_FAILED`].[LogicalResourceId,ResourceStatusReason]' \
  --output table
```

### Issue: Website not accessible

**Solution:**
```bash
# Check security group
aws ec2 describe-security-groups \
  --filters "Name=tag:Name,Values=DailyAgenda-WebServer-SG" \
  --query 'SecurityGroups[0].IpPermissions'

# Check instance status
aws ec2 describe-instances \
  --filters "Name=tag:Name,Values=daily-agenda-service" \
  --query 'Reservations[0].Instances[0].[State.Name,PublicIpAddress]'
```

---

## 📚 Full Documentation

- **[Complete README](../README.md)** - Full project documentation
- **[Deployment Guide](../docs/deployment-guide.md)** - Detailed deployment steps
- **[Architecture Explanation](../docs/architecture-explanation.md)** - In-depth architecture details

---

## 💬 Need Help?

- Open an issue: https://github.com/rman90/AWS-Projects/issues
- Check AWS CloudFormation docs: https://docs.aws.amazon.com/cloudformation/

---

**Built with ☁️ by Ross Nesbitt**
