# 🏗️ Architecture Explanation - Daily Agenda Infrastructure

This document provides an in-depth explanation of the AWS infrastructure architecture for the Daily Agenda application.

---

## 📋 Table of Contents

1. [Architecture Overview](#architecture-overview)
2. [Network Architecture](#network-architecture)
3. [Compute Architecture](#compute-architecture)
4. [Security Architecture](#security-architecture)
5. [Application Architecture](#application-architecture)
6. [Nested Stack Design](#nested-stack-design)
7. [Data Flow](#data-flow)
8. [High Availability Considerations](#high-availability-considerations)
9. [Cost Optimization](#cost-optimization)
10. [Scalability Path](#scalability-path)

---

## Architecture Overview

### Design Philosophy

The Daily Agenda infrastructure follows AWS Well-Architected Framework principles:

- **Operational Excellence:** Infrastructure as Code enables consistent deployments
- **Security:** Least-privilege access, security groups, and VPC isolation
- **Reliability:** Automated provisioning and configuration management
- **Performance Efficiency:** Right-sized resources for the workload
- **Cost Optimization:** Use of free-tier eligible resources where possible

### Architecture Layers

```
┌─────────────────────────────────────────────────────────┐
│                    Presentation Layer                    │
│              (Web Browser / HTTP Client)                 │
└────────────────────────┬────────────────────────────────┘
                         │ HTTP (Port 80)
┌────────────────────────▼────────────────────────────────┐
│                     Internet Layer                       │
│                  (Internet Gateway)                      │
└────────────────────────┬────────────────────────────────┘
                         │
┌────────────────────────▼────────────────────────────────┐
│                    Network Layer                         │
│         (VPC, Subnets, Route Tables, NACLs)             │
└────────────────────────┬────────────────────────────────┘
                         │
┌────────────────────────▼────────────────────────────────┐
│                    Security Layer                        │
│                  (Security Groups)                       │
└────────────────────────┬────────────────────────────────┘
                         │
┌────────────────────────▼────────────────────────────────┐
│                    Compute Layer                         │
│              (EC2 Instance, EBS Volume)                  │
└────────────────────────┬────────────────────────────────┘
                         │
┌────────────────────────▼────────────────────────────────┐
│                   Application Layer                      │
│         (Apache, PHP, Daily Agenda Website)             │
└────────────────────────┬────────────────────────────────┘
                         │
┌────────────────────────▼────────────────────────────────┐
│                  Configuration Layer                     │
│              (SSM Parameter Store)                       │
└─────────────────────────────────────────────────────────┘
```

---

## Network Architecture

### VPC Design

**CIDR Block:** 10.0.0.0/16 (65,536 IP addresses)

**Key Features:**
- DNS resolution enabled (`EnableDnsSupport: true`)
- DNS hostnames enabled (`EnableDnsHostnames: true`)
- Single VPC for simplicity and cost optimization

**Why this CIDR?**
- RFC 1918 private address space
- Large enough for future expansion
- Avoids conflicts with common corporate networks

### Subnet Architecture

**Public Subnet:** 10.0.0.0/24 (256 IP addresses)

**Characteristics:**
- Deployed in first Availability Zone
- Auto-assign public IPv4 addresses enabled
- Associated with public route table
- Direct route to Internet Gateway

**Design Rationale:**
- Single subnet sufficient for demo/development
- Production would use multiple AZs for high availability
- /24 provides adequate IP space for small deployments

### Internet Connectivity

**Internet Gateway (IGW):**
- Attached to VPC
- Enables bidirectional internet communication
- Horizontally scaled, redundant, and highly available (AWS-managed)

**Route Table Configuration:**

| Destination | Target | Purpose |
|-------------|--------|---------|
| 10.0.0.0/16 | local | Intra-VPC communication |
| 0.0.0.0/0 | igw-xxxxx | Internet-bound traffic |

### Network ACLs

**Default NACL:** Used for simplicity
- Allows all inbound traffic
- Allows all outbound traffic
- Stateless firewall at subnet level

**Production Recommendation:**
- Create custom NACLs with explicit allow rules
- Deny rules for known malicious IPs
- Separate NACLs for different subnet tiers

---

## Compute Architecture

### EC2 Instance Specifications

**Instance Type:** t2.micro
- 1 vCPU
- 1 GB RAM
- Low to moderate network performance
- Free tier eligible (750 hours/month for 12 months)

**Operating System:** Amazon Linux 2
- AWS-optimized Linux distribution
- Long-term support
- Pre-installed AWS tools (CLI, SSM agent)
- Regular security updates

**Why t2.micro?**
- Cost-effective for low-traffic applications
- Sufficient for demo/development workloads
- Burstable performance for occasional traffic spikes
- Free tier eligible

### Storage Architecture

**Root Volume:**
- EBS General Purpose SSD (gp2)
- Size determined by AMI (typically 8 GB)
- Deleted on instance termination

**Additional Data Volume:**
- 100 GB EBS General Purpose SSD (gp2)
- Attached as /dev/sdh
- Demonstrates EBS volume management
- Deleted on stack deletion (`DeletionPolicy: Delete`)

**Storage Use Cases:**
- Root volume: OS and application files
- Data volume: Logs, backups, or application data

### IAM Instance Profile

**Role:** ec2ssm

**Permissions:**
- `AmazonSSMManagedInstanceCore`: Systems Manager access
- `AmazonSSMReadOnlyAccess`: Read SSM Parameter Store

**Why SSM?**
- Secure parameter storage without hardcoding
- No need to manage SSH keys
- Audit trail of parameter access
- Integration with AWS Secrets Manager

---

## Security Architecture

### Defense in Depth

```
Internet
   │
   ├─► Security Group (Stateful Firewall)
   │      │
   │      ├─► Ingress: HTTP (80) from 0.0.0.0/0
   │      └─► Egress: HTTP (80), HTTPS (443) to 0.0.0.0/0
   │
   ├─► Network ACL (Stateless Firewall)
   │      │
   │      └─► Default: Allow all (can be restricted)
   │
   └─► EC2 Instance
          │
          ├─► IAM Role (Least Privilege)
          │      │
          │      └─► SSM Parameter Store access only
          │
          └─► Application (Apache + PHP)
```

### Security Group Configuration

**Inbound Rules:**

| Protocol | Port | Source | Purpose |
|----------|------|--------|---------|
| TCP | 80 | 0.0.0.0/0 | HTTP web traffic |

**Outbound Rules:**

| Protocol | Port | Destination | Purpose |
|----------|------|-------------|---------|
| TCP | 80 | 0.0.0.0/0 | HTTP for package updates |
| TCP | 443 | 0.0.0.0/0 | HTTPS for AWS API calls |

**Security Considerations:**
- ✅ No SSH access (use SSM Session Manager instead)
- ✅ Restricted egress (only HTTP/HTTPS)
- ✅ Stateful firewall (return traffic automatically allowed)
- ⚠️ HTTP from 0.0.0.0/0 (acceptable for public website)

### Production Security Enhancements

**Recommended Improvements:**

1. **HTTPS/TLS:**
   - Add Application Load Balancer
   - Use ACM for SSL/TLS certificates
   - Redirect HTTP to HTTPS

2. **Network Segmentation:**
   - Private subnets for application tier
   - Public subnets only for load balancers
   - NAT Gateway for outbound internet access

3. **Web Application Firewall:**
   - AWS WAF for OWASP Top 10 protection
   - Rate limiting
   - Geo-blocking if needed

4. **Monitoring & Logging:**
   - VPC Flow Logs
   - CloudWatch Logs for application logs
   - CloudTrail for API audit logs
   - GuardDuty for threat detection

5. **Secrets Management:**
   - AWS Secrets Manager for sensitive data
   - Rotate credentials automatically
   - Encrypt at rest with KMS

---

## Application Architecture

### Web Server Stack

**Apache HTTP Server (httpd):**
- Industry-standard web server
- Lightweight and performant
- Extensive module ecosystem
- Well-documented and supported

**PHP Runtime:**
- Server-side scripting language
- Integrates with Apache via mod_php
- Executes AWS CLI commands for SSM access
- Simple and effective for this use case

### Application Components

**1. index.html (Frontend)**
```
┌─────────────────────────────────────┐
│         HTML Structure              │
│  ┌───────────────────────────────┐  │
│  │  Banner Image (S3-hosted)     │  │
│  └───────────────────────────────┘  │
│  ┌───────────────────────────────┐  │
│  │  "Daily Agenda" Heading       │  │
│  └───────────────────────────────┘  │
│  ┌───────────────────────────────┐  │
│  │  JavaScript Fetch API         │  │
│  │  ├─► GET /content.php         │  │
│  │  └─► Render agenda items      │  │
│  └───────────────────────────────┘  │
└─────────────────────────────────────┘
```

**2. content.php (Backend)**
```
┌─────────────────────────────────────┐
│         PHP Script                  │
│  ┌───────────────────────────────┐  │
│  │  getSSMParameterValue()       │  │
│  │  ├─► Execute AWS CLI          │  │
│  │  ├─► Query SSM Parameter      │  │
│  │  └─► Return value             │  │
│  └───────────────────────────────┘  │
│  ┌───────────────────────────────┐  │
│  │  Parse comma-separated list   │  │
│  └───────────────────────────────┘  │
│  ┌───────────────────────────────┐  │
│  │  Generate HTML list           │  │
│  └───────────────────────────────┘  │
└─────────────────────────────────────┘
```

### Bootstrap Process

**CloudFormation Init (cfn-init):**

1. **Environment Variables:**
   - Set AWS region for AWS CLI commands

2. **Package Installation:**
   - Update system packages
   - Install Apache (httpd)
   - Install PHP runtime

3. **Configuration:**
   - Set region environment variable in Apache config
   - Enable Apache to start on boot

4. **File Creation:**
   - Deploy index.html
   - Deploy content.php
   - Set proper ownership and permissions

5. **Service Management:**
   - Start Apache service
   - Enable Apache on boot

**UserData Script:**
- Install aws-cfn-bootstrap tools
- Install PHP
- Execute cfn-init
- Signal CloudFormation on completion

---

## Nested Stack Design

### Why Nested Stacks?

**Benefits:**

1. **Modularity:**
   - Separate concerns (network, compute, application)
   - Each stack has a single responsibility
   - Easier to understand and maintain

2. **Reusability:**
   - Network stack can be reused across projects
   - Standard patterns become templates
   - Reduce code duplication

3. **Team Collaboration:**
   - Network team owns network stack
   - Application team owns app stack
   - Clear boundaries and ownership

4. **Change Management:**
   - Update one stack without affecting others
   - Reduced blast radius for changes
   - Easier rollback of specific components

5. **Scalability:**
   - Add new stacks (database, caching, monitoring)
   - Compose complex architectures from simple building blocks
   - Manage large infrastructures effectively

### Stack Hierarchy

```
Parent Stack (Orchestration)
│
├─► Network Stack (Infrastructure)
│   │
│   ├─► VPC
│   ├─► Internet Gateway
│   ├─► Subnets
│   ├─► Route Tables
│   └─► Network ACLs
│   │
│   └─► Outputs:
│       ├─► VPC ID
│       └─► Subnet ID
│
└─► Application Stack (Compute)
    │
    ├─► Inputs:
    │   ├─► VPC ID (from Network Stack)
    │   └─► Subnet ID (from Network Stack)
    │
    ├─► Security Group
    ├─► EC2 Instance
    ├─► EBS Volume
    └─► SSM Parameter
    │
    └─► Outputs:
        └─► Website URL
```

### Cross-Stack References

**Mechanism:** CloudFormation Outputs and GetAtt

**Example Flow:**
1. Network Stack creates VPC
2. Network Stack exports VPC ID as output
3. Parent Stack retrieves VPC ID using `!GetAtt`
4. Parent Stack passes VPC ID to Application Stack as parameter
5. Application Stack uses VPC ID to create Security Group

**Code Example:**
```yaml
# Network Stack Output
Outputs:
  VPC:
    Value: !Ref VPC
    Export:
      Name: !Sub '${AWS::StackName}-VPC'

# Parent Stack Reference
LabApplication:
  Properties:
    Parameters:
      VpcId: !GetAtt 'LabNetwork.Outputs.VPC'

# Application Stack Parameter
Parameters:
  VpcId:
    Type: AWS::EC2::VPC::Id
```

### Template Storage

**S3 Bucket Requirements:**
- Nested templates must be in S3
- Bucket must be in same region as stack
- CloudFormation needs read access
- Versioning recommended for change tracking

**Naming Convention:**
- `templates-${AWS::AccountId}` ensures uniqueness
- Prevents conflicts across accounts
- Easy to identify purpose

---

## Data Flow

### Request Flow

```
1. User Browser
   │
   ├─► HTTP GET http://ec2-xxx.compute.amazonaws.com/
   │
2. Internet Gateway
   │
   ├─► Routes to VPC
   │
3. Security Group
   │
   ├─► Allows port 80 inbound
   │
4. EC2 Instance (Apache)
   │
   ├─► Serves index.html
   │
5. Browser JavaScript
   │
   ├─► Fetch /content.php
   │
6. Apache + PHP
   │
   ├─► Execute content.php
   │
7. AWS CLI (via IAM role)
   │
   ├─► aws ssm get-parameter --name /daily_agenda
   │
8. SSM Parameter Store
   │
   ├─► Returns: "Item1,Item2,Item3"
   │
9. PHP Processing
   │
   ├─► Parse comma-separated values
   ├─► Generate HTML list
   │
10. HTTP Response
    │
    └─► Returns HTML to browser
```

### Configuration Flow

```
CloudFormation Template
   │
   ├─► Creates SSM Parameter
   │      │
   │      └─► Stores: "Team standup,Code review,Deploy"
   │
   ├─► Creates EC2 Instance
   │      │
   │      ├─► UserData script runs
   │      │      │
   │      │      └─► Installs Apache, PHP
   │      │
   │      └─► cfn-init runs
   │             │
   │             ├─► Creates index.html
   │             ├─► Creates content.php
   │             └─► Starts Apache
   │
   └─► Application Ready
```

---

## High Availability Considerations

### Current Architecture Limitations

**Single Points of Failure:**
- Single EC2 instance
- Single Availability Zone
- No redundancy

**Downtime Scenarios:**
- AZ failure
- Instance failure
- Maintenance windows

### High Availability Design

**Recommended Architecture:**

```
                    Internet
                       │
                       ▼
              Application Load Balancer
                (Multi-AZ)
                       │
        ┌──────────────┼──────────────┐
        │              │              │
        ▼              ▼              ▼
    EC2 Instance   EC2 Instance   EC2 Instance
    (AZ-1a)        (AZ-1b)        (AZ-1c)
        │              │              │
        └──────────────┴──────────────┘
                       │
                       ▼
              Auto Scaling Group
              (Min: 2, Max: 6)
```

**Components:**

1. **Application Load Balancer:**
   - Distributes traffic across instances
   - Health checks and automatic failover
   - SSL/TLS termination

2. **Auto Scaling Group:**
   - Maintains desired instance count
   - Replaces unhealthy instances
   - Scales based on demand

3. **Multi-AZ Deployment:**
   - Instances in multiple Availability Zones
   - Survives AZ failures
   - Reduced latency for users

4. **RDS Database (if needed):**
   - Multi-AZ deployment
   - Automated backups
   - Read replicas for scaling

---

## Cost Optimization

### Current Monthly Costs (Estimated)

| Resource | Quantity | Unit Cost | Monthly Cost |
|----------|----------|-----------|--------------|
| t2.micro EC2 | 1 | $0.0116/hr | $8.47 |
| EBS gp2 (100 GB) | 1 | $0.10/GB | $10.00 |
| EBS gp2 (8 GB root) | 1 | $0.10/GB | $0.80 |
| Data Transfer | ~10 GB | $0.09/GB | $0.90 |
| **Total** | | | **~$20/month** |

**Free Tier Benefits:**
- t2.micro: 750 hours/month free (first 12 months)
- EBS: 30 GB free (first 12 months)
- Data Transfer: 1 GB/month free (always)

**Actual Cost with Free Tier:** ~$0-5/month

### Cost Optimization Strategies

1. **Right-Sizing:**
   - Monitor CPU/memory utilization
   - Downsize if underutilized
   - Use Compute Optimizer recommendations

2. **Reserved Instances:**
   - 1-year or 3-year commitments
   - Up to 72% savings vs. on-demand
   - Suitable for production workloads

3. **Spot Instances:**
   - Up to 90% savings
   - Suitable for fault-tolerant workloads
   - Use with Auto Scaling Groups

4. **Storage Optimization:**
   - Use gp3 instead of gp2 (20% cheaper)
   - Delete unused EBS snapshots
   - Use S3 for static content

5. **Scheduled Scaling:**
   - Stop instances during off-hours
   - Use Lambda to automate start/stop
   - Ideal for dev/test environments

---

## Scalability Path

### Vertical Scaling (Scale Up)

**Upgrade Instance Type:**
```
t2.micro → t2.small → t2.medium → t3.large
```

**Benefits:**
- Simple to implement
- No architecture changes
- Immediate performance improvement

**Limitations:**
- Downtime during resize
- Upper limits on instance size
- Single point of failure remains

### Horizontal Scaling (Scale Out)

**Add More Instances:**

**Phase 1: Manual Scaling**
- Deploy additional EC2 instances
- Add Application Load Balancer
- Distribute traffic manually

**Phase 2: Auto Scaling**
- Create Auto Scaling Group
- Define scaling policies
- Automatic scale out/in based on metrics

**Phase 3: Multi-Region**
- Deploy in multiple AWS regions
- Use Route 53 for global load balancing
- Reduce latency for global users

### Database Tier (Future)

**Current:** SSM Parameter Store (simple key-value)

**Scalability Path:**
1. **Amazon RDS:** Relational database for structured data
2. **Amazon DynamoDB:** NoSQL for high-scale, low-latency
3. **Amazon ElastiCache:** In-memory caching layer
4. **Amazon Aurora:** MySQL/PostgreSQL compatible, auto-scaling

### Caching Strategy

**CloudFront CDN:**
- Cache static content globally
- Reduce origin load
- Improve user experience

**ElastiCache:**
- Cache database queries
- Session storage
- Reduce database load

---

## Conclusion

This architecture demonstrates fundamental AWS infrastructure patterns suitable for small to medium web applications. The modular design using nested stacks provides a foundation for growth, while the current implementation remains cost-effective and simple to manage.

**Key Takeaways:**
- Infrastructure as Code enables consistent, repeatable deployments
- Nested stacks provide modularity and maintainability
- Security is implemented at multiple layers
- The architecture can scale both vertically and horizontally
- Cost optimization is achievable through right-sizing and AWS free tier

**Next Steps:**
- Implement high availability with ALB and Auto Scaling
- Add HTTPS/TLS for secure communication
- Integrate with CI/CD pipeline for automated deployments
- Add monitoring and alerting with CloudWatch
- Implement backup and disaster recovery strategies

---

**Questions?** Open an issue on [GitHub](https://github.com/rman90/AWS-Projects/issues)
