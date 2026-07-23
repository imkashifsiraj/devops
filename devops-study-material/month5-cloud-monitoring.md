# Month 5: Cloud Computing & Monitoring

## DevOps Upskilling Program — GCC Job Market Focus

> **Duration:** 4-6 weeks | **Prerequisites:** Month 1-4 (Linux, Networking, Git, CI/CD, Containers)
> **Focus:** AWS + Azure (Azure dominant in GCC) + Monitoring & Observability

---

## Table of Contents

1. [Cloud Computing Fundamentals](#section-1-cloud-computing-fundamentals)
2. [AWS Cloud](#section-2-aws-cloud)
3. [Azure Cloud](#section-3-azure-cloud)
4. [Monitoring & Observability](#section-4-monitoring--observability)
5. [Best Practices](#section-5-best-practices)
6. [Lessons Learned / Pro Tips](#section-6-lessons-learned--pro-tips)
7. [Interview Questions](#section-7-interview-questions)
8. [Free Resources](#section-8-free-resources-with-links)
9. [Certification Prep](#section-9-certification-prep)

---

## Section 1: Cloud Computing Fundamentals

### What is Cloud Computing?

Cloud computing is the on-demand delivery of IT resources over the internet with pay-as-you-go pricing. Instead of owning physical data centers and servers, you rent access to computing power, storage, and databases from a cloud provider.

#### Service Models

| Model | You Manage | Provider Manages | Examples |
|-------|-----------|-----------------|----------|
| **IaaS** (Infrastructure as a Service) | OS, Runtime, Apps, Data | Hardware, Networking, Virtualization | EC2, Azure VMs, GCE |
| **PaaS** (Platform as a Service) | Apps, Data | OS, Runtime, Hardware, Networking | Elastic Beanstalk, Azure App Service, Heroku |
| **SaaS** (Software as a Service) | Just use it | Everything | Office 365, Salesforce, Gmail |

**Memory Aid:** IaaS = "I manage Almost all Software" | PaaS = "Provider manages Almost all Systems" | SaaS = "Someone else manages ALL"

### Cloud Benefits

1. **Scalability** — Scale up/down based on demand (horizontal & vertical)
2. **Cost Efficiency** — No upfront capital expenditure (CapEx → OpEx)
3. **Global Reach** — Deploy in multiple regions worldwide in minutes
4. **High Availability** — Built-in redundancy across availability zones
5. **Elasticity** — Auto-scale resources to match demand
6. **Speed & Agility** — Provision resources in minutes, not weeks
7. **Security** — Cloud providers invest billions in security infrastructure
8. **Disaster Recovery** — Built-in backup and recovery mechanisms

### Shared Responsibility Model

```
┌─────────────────────────────────────────────────────────────┐
│                    CUSTOMER RESPONSIBILITY                     │
│  Data, Identity & Access, Application, OS (IaaS),            │
│  Network & Firewall config, Client-side encryption           │
├─────────────────────────────────────────────────────────────┤
│                    PROVIDER RESPONSIBILITY                     │
│  Physical security, Hardware, Networking, Virtualization,     │
│  Storage infrastructure, Global infrastructure               │
└─────────────────────────────────────────────────────────────┘
```

**Key Principle:** Security OF the cloud (provider) vs Security IN the cloud (customer)

### Well-Architected Framework (6 Pillars)

| Pillar | Focus | Key Practices |
|--------|-------|---------------|
| **Operational Excellence** | Run & monitor systems | Automate, anticipate failure, learn from events |
| **Security** | Protect data & systems | Least privilege, encryption, traceability |
| **Reliability** | Recover from failure | Auto-scale, distributed design, test recovery |
| **Performance Efficiency** | Use resources efficiently | Right-size, monitor, use managed services |
| **Cost Optimization** | Avoid unnecessary costs | Pay for what you use, reserved instances |
| **Sustainability** | Minimize environmental impact | Right-size, efficient code, managed services |

### Multi-Cloud and Hybrid Cloud

**Multi-Cloud:** Using multiple cloud providers (e.g., AWS + Azure)
- Avoids vendor lock-in
- Best-of-breed services
- Regulatory compliance across regions
- Common in GCC where government mandates may require specific providers

**Hybrid Cloud:** Combining on-premises with cloud
- Sensitive data stays on-premises
- Burst to cloud for peak demand
- Gradual migration path
- Common in GCC banking/government sectors

### Cloud Cost Optimization Strategies

1. **Right-sizing** — Match instance types to actual workload needs
2. **Reserved Instances / Savings Plans** — Commit for 1-3 years for up to 72% discount
3. **Spot Instances / Spot VMs** — Up to 90% discount for interruptible workloads
4. **Auto-scaling** — Scale down during off-peak hours
5. **Storage tiering** — Move infrequently accessed data to cheaper tiers
6. **Delete unused resources** — Unattached volumes, idle load balancers
7. **Scheduled start/stop** — Dev/test environments off at night
8. **Tagging strategy** — Track costs by project, team, environment
9. **Use managed services** — Often cheaper than self-managed at scale
10. **Monitor with budgets/alerts** — AWS Budgets, Azure Cost Management

### GCC Cloud Landscape

**Azure in GCC:**
- **UAE North (Dubai)** — Primary region, launched 2019
- **UAE Central (Abu Dhabi)** — Paired region
- **Qatar Central (Doha)** — Launched 2022
- **Saudi Arabia (Jeddah planned)** — Expanding
- Azure is the dominant cloud provider in GCC government and enterprise

**AWS in GCC:**
- **Bahrain (me-south-1)** — Launched 2019, 3 AZs
- **UAE (me-central-1)** — Launched 2022, 3 AZs
- **Israel (il-central-1)** — 2023
- Growing adoption in startups and tech companies

**Data Residency Requirements:**
- Saudi Arabia: NDMO requires data to stay within KSA for government
- UAE: Central Bank data must remain in UAE
- Many GCC countries have evolving data sovereignty laws
- Both Azure and AWS offer government-specific clouds and compliance

**Government Cloud Options:**
- AWS GovCloud (US-specific, but model applies)
- Azure Government (similar model)
- Both providers offer dedicated regions with enhanced compliance

---

## Section 2: AWS Cloud

### 2.1 AWS Account & IAM

#### AWS Account Structure

```
┌─────────────────────────────────────────────┐
│           AWS Organizations                   │
│  ┌─────────────────────────────────────┐    │
│  │  Management Account (Root)           │    │
│  │  ┌───────────┐  ┌───────────┐      │    │
│  │  │  OU:Prod  │  │  OU:Dev   │      │    │
│  │  │ ┌───────┐ │  │ ┌───────┐ │      │    │
│  │  │ │Acct 1 │ │  │ │Acct 3 │ │      │    │
│  │  │ │Acct 2 │ │  │ │Acct 4 │ │      │    │
│  │  │ └───────┘ │  │ └───────┘ │      │    │
│  │  └───────────┘  └───────────┘      │    │
│  └─────────────────────────────────────┘    │
└─────────────────────────────────────────────┘
```

- **Organizations:** Central management of multiple AWS accounts
- **Organizational Units (OUs):** Group accounts by function (Prod, Dev, Security)
- **Service Control Policies (SCPs):** Guardrails applied at OU/account level
- **Consolidated Billing:** Single bill for all accounts

```bash
# List accounts in organization
aws organizations list-accounts

# List organizational units
aws organizations list-organizational-units-for-parent --parent-id r-xxxx

# Create an account
aws organizations create-account --email new@company.com --account-name "Production"
```

#### IAM Users, Groups, Roles, Policies

**Users:** Individual identities with credentials
**Groups:** Collection of users (attach policies to groups, not users)
**Roles:** Temporary credentials assumed by services/users
**Policies:** JSON documents defining permissions

```bash
# Create a user
aws iam create-user --user-name devops-engineer

# Create a group and add user
aws iam create-group --group-name DevOpsTeam
aws iam add-user-to-group --user-name devops-engineer --group-name DevOpsTeam

# Attach policy to group
aws iam attach-group-policy --group-name DevOpsTeam \
  --policy-arn arn:aws:iam::aws:policy/AmazonEC2FullAccess

# Create access keys for CLI
aws iam create-access-key --user-name devops-engineer
```

#### IAM Policy JSON Structure

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowEC2Describe",
      "Effect": "Allow",
      "Action": [
        "ec2:Describe*",
        "ec2:List*"
      ],
      "Resource": "*",
      "Condition": {
        "StringEquals": {
          "aws:RequestedRegion": "me-south-1"
        }
      }
    },
    {
      "Sid": "DenyDeleteProduction",
      "Effect": "Deny",
      "Action": "ec2:TerminateInstances",
      "Resource": "*",
      "Condition": {
        "StringEquals": {
          "ec2:ResourceTag/Environment": "production"
        }
      }
    }
  ]
}
```

**Policy Elements:**
- `Version` — Always "2012-10-17" (latest)
- `Statement` — Array of permission statements
- `Sid` — Optional identifier for the statement
- `Effect` — "Allow" or "Deny" (Deny always wins)
- `Action` — API actions (supports wildcards)
- `Resource` — ARN of resources (supports wildcards)
- `Condition` — Optional conditions (IP, time, tags, region)

#### IAM Best Practices

1. Never use root account for daily tasks
2. Enable MFA on root and all users
3. Use roles instead of long-term access keys
4. Apply least privilege principle
5. Use groups to assign permissions
6. Rotate credentials regularly
7. Use IAM Access Analyzer to find unused permissions
8. Use SCPs to enforce guardrails across accounts
9. Enable CloudTrail for audit logging
10. Use conditions to restrict by IP, region, or MFA

#### STS and AssumeRole

```bash
# Assume a role (returns temporary credentials)
aws sts assume-role \
  --role-arn arn:aws:iam::123456789012:role/DevOpsRole \
  --role-session-name "deploy-session"

# Response contains: AccessKeyId, SecretAccessKey, SessionToken

# Get current identity
aws sts get-caller-identity
```

**Cross-account access pattern:**
1. Account B creates role with trust policy for Account A
2. User in Account A assumes role in Account B
3. Temporary credentials returned (valid 1-12 hours)

#### Service-Linked Roles

Pre-defined roles that AWS services use to perform actions:
```bash
# List service-linked roles
aws iam list-roles --query "Roles[?starts_with(Path, '/aws-service-role/')]"

# Examples:
# AWSServiceRoleForAutoScaling — used by Auto Scaling
# AWSServiceRoleForECS — used by ECS
# AWSServiceRoleForElasticLoadBalancing — used by ELB
```

#### Instance Profiles

An instance profile is a container for an IAM role attached to EC2:
```bash
# Create role for EC2
aws iam create-role --role-name EC2-S3-Access \
  --assume-role-policy-document '{
    "Version": "2012-10-17",
    "Statement": [{
      "Effect": "Allow",
      "Principal": {"Service": "ec2.amazonaws.com"},
      "Action": "sts:AssumeRole"
    }]
  }'

# Attach S3 policy
aws iam attach-role-policy --role-name EC2-S3-Access \
  --policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess

# Create instance profile and add role
aws iam create-instance-profile --instance-profile-name EC2-S3-Profile
aws iam add-role-to-instance-profile \
  --instance-profile-name EC2-S3-Profile \
  --role-name EC2-S3-Access
```

#### CLI Configuration

```bash
# Basic configuration
aws configure
# Enter: Access Key, Secret Key, Region (me-south-1), Output (json)

# Named profiles
aws configure --profile production
aws configure --profile development

# Use a profile
aws s3 ls --profile production

# AWS SSO configuration
aws configure sso
# Enter: SSO start URL, SSO region, account, role

# Environment variables (override config)
export AWS_ACCESS_KEY_ID=AKIAIOSFODNN7EXAMPLE
export AWS_SECRET_ACCESS_KEY=wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
export AWS_DEFAULT_REGION=me-south-1

# Config file locations
# ~/.aws/credentials — access keys
# ~/.aws/config — region, output, profiles
```

---

### 2.2 Networking (VPC)

#### VPC, Subnets, and CIDR Planning

```bash
# Create a VPC
aws ec2 create-vpc --cidr-block 10.0.0.0/16 --tag-specifications \
  'ResourceType=vpc,Tags=[{Key=Name,Value=production-vpc}]'

# Create subnets
aws ec2 create-subnet --vpc-id vpc-xxx --cidr-block 10.0.1.0/24 \
  --availability-zone me-south-1a --tag-specifications \
  'ResourceType=subnet,Tags=[{Key=Name,Value=public-subnet-1a}]'

aws ec2 create-subnet --vpc-id vpc-xxx --cidr-block 10.0.2.0/24 \
  --availability-zone me-south-1b --tag-specifications \
  'ResourceType=subnet,Tags=[{Key=Name,Value=public-subnet-1b}]'

aws ec2 create-subnet --vpc-id vpc-xxx --cidr-block 10.0.10.0/24 \
  --availability-zone me-south-1a --tag-specifications \
  'ResourceType=subnet,Tags=[{Key=Name,Value=private-subnet-1a}]'

aws ec2 create-subnet --vpc-id vpc-xxx --cidr-block 10.0.11.0/24 \
  --availability-zone me-south-1b --tag-specifications \
  'ResourceType=subnet,Tags=[{Key=Name,Value=private-subnet-1b}]'
```

**CIDR Planning Best Practice:**
| Subnet Type | CIDR | IPs Available | Purpose |
|-------------|------|---------------|---------|
| Public 1a | 10.0.1.0/24 | 251 | Load Balancers, Bastion |
| Public 1b | 10.0.2.0/24 | 251 | Load Balancers, NAT GW |
| Private App 1a | 10.0.10.0/24 | 251 | Application servers |
| Private App 1b | 10.0.11.0/24 | 251 | Application servers |
| Private DB 1a | 10.0.20.0/24 | 251 | Databases |
| Private DB 1b | 10.0.21.0/24 | 251 | Databases |

#### Internet Gateway & NAT Gateway

```bash
# Create and attach Internet Gateway (for public subnets)
aws ec2 create-internet-gateway --tag-specifications \
  'ResourceType=internet-gateway,Tags=[{Key=Name,Value=prod-igw}]'
aws ec2 attach-internet-gateway --internet-gateway-id igw-xxx --vpc-id vpc-xxx

# Create NAT Gateway (for private subnet internet access)
# First, allocate an Elastic IP
aws ec2 allocate-address --domain vpc
# Then create NAT Gateway in PUBLIC subnet
aws ec2 create-nat-gateway --subnet-id subnet-public-xxx \
  --allocation-id eipalloc-xxx --tag-specifications \
  'ResourceType=natgateway,Tags=[{Key=Name,Value=prod-nat}]'
```

#### Route Tables

```bash
# Create route table for public subnets
aws ec2 create-route-table --vpc-id vpc-xxx
aws ec2 create-route --route-table-id rtb-xxx \
  --destination-cidr-block 0.0.0.0/0 --gateway-id igw-xxx
aws ec2 associate-route-table --route-table-id rtb-xxx --subnet-id subnet-public-xxx

# Create route table for private subnets
aws ec2 create-route-table --vpc-id vpc-xxx
aws ec2 create-route --route-table-id rtb-yyy \
  --destination-cidr-block 0.0.0.0/0 --nat-gateway-id nat-xxx
aws ec2 associate-route-table --route-table-id rtb-yyy --subnet-id subnet-private-xxx
```

#### Security Groups vs NACLs

| Feature | Security Groups | NACLs |
|---------|----------------|-------|
| Level | Instance (ENI) level | Subnet level |
| State | **Stateful** (return traffic auto-allowed) | **Stateless** (must allow return traffic) |
| Rules | Allow rules only | Allow AND Deny rules |
| Evaluation | All rules evaluated together | Rules processed in order (number) |
| Default | Deny all inbound, Allow all outbound | Allow all inbound and outbound |
| Association | Multiple SGs per instance | One NACL per subnet |

```bash
# Create Security Group
aws ec2 create-security-group --group-name web-sg \
  --description "Web server security group" --vpc-id vpc-xxx

# Allow HTTP/HTTPS inbound
aws ec2 authorize-security-group-ingress --group-id sg-xxx \
  --protocol tcp --port 80 --cidr 0.0.0.0/0
aws ec2 authorize-security-group-ingress --group-id sg-xxx \
  --protocol tcp --port 443 --cidr 0.0.0.0/0

# Allow SSH from specific IP only
aws ec2 authorize-security-group-ingress --group-id sg-xxx \
  --protocol tcp --port 22 --cidr 10.0.0.0/16

# Create NACL
aws ec2 create-network-acl --vpc-id vpc-xxx
aws ec2 create-network-acl-entry --network-acl-id acl-xxx \
  --rule-number 100 --protocol tcp --port-range From=80,To=80 \
  --cidr-block 0.0.0.0/0 --rule-action allow --ingress
```

#### VPC Peering & Transit Gateway

```bash
# VPC Peering — connects two VPCs directly
aws ec2 create-vpc-peering-connection \
  --vpc-id vpc-xxx --peer-vpc-id vpc-yyy --peer-region me-south-1

# Accept peering connection
aws ec2 accept-vpc-peering-connection --vpc-peering-connection-id pcx-xxx

# Transit Gateway — hub-and-spoke for multiple VPCs
aws ec2 create-transit-gateway --description "Central TGW" \
  --options DefaultRouteTableAssociation=enable
```

**When to use what:**
- **VPC Peering:** 2-3 VPCs, simple connectivity, no transitive routing
- **Transit Gateway:** Many VPCs, hub-and-spoke, transitive routing, centralized management

#### VPC Endpoints

```bash
# Gateway Endpoint (S3, DynamoDB) — FREE
aws ec2 create-vpc-endpoint --vpc-id vpc-xxx \
  --service-name com.amazonaws.me-south-1.s3 \
  --route-table-ids rtb-xxx

# Interface Endpoint (most other services) — costs per hour + data
aws ec2 create-vpc-endpoint --vpc-id vpc-xxx \
  --vpc-endpoint-type Interface \
  --service-name com.amazonaws.me-south-1.ecr.api \
  --subnet-ids subnet-xxx --security-group-ids sg-xxx
```

#### VPN and Direct Connect

- **Site-to-Site VPN:** Encrypted tunnel over internet (quick to set up, variable latency)
- **Direct Connect:** Dedicated physical connection (consistent latency, higher bandwidth)
- **Direct Connect in GCC:** Available in Bahrain and UAE through partners

#### Complete VPC Architecture (ASCII)

```
┌─────────────────────────────── VPC 10.0.0.0/16 ──────────────────────────────────┐
│                                                                                     │
│  ┌──── AZ-1a ────────────────────┐    ┌──── AZ-1b ────────────────────┐          │
│  │                                │    │                                │          │
│  │  ┌─ Public 10.0.1.0/24 ────┐  │    │  ┌─ Public 10.0.2.0/24 ────┐  │          │
│  │  │  [ALB]  [NAT GW]        │  │    │  │  [ALB]  [Bastion]       │  │          │
│  │  └─────────────────────────┘  │    │  └─────────────────────────┘  │          │
│  │                                │    │                                │          │
│  │  ┌─ Private 10.0.10.0/24 ──┐  │    │  ┌─ Private 10.0.11.0/24 ──┐  │          │
│  │  │  [EC2 App] [ECS Tasks]  │  │    │  │  [EC2 App] [ECS Tasks]  │  │          │
│  │  └─────────────────────────┘  │    │  └─────────────────────────┘  │          │
│  │                                │    │                                │          │
│  │  ┌─ Private 10.0.20.0/24 ──┐  │    │  ┌─ Private 10.0.21.0/24 ──┐  │          │
│  │  │  [RDS Primary]           │  │    │  │  [RDS Standby]          │  │          │
│  │  └─────────────────────────┘  │    │  └─────────────────────────┘  │          │
│  │                                │    │                                │          │
│  └────────────────────────────────┘    └────────────────────────────────┘          │
│                                                                                     │
│  [IGW] ←→ Internet        [VPC Endpoint] ←→ S3        [NAT GW] ←→ Internet       │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

---

### 2.3 Compute

#### EC2 (Elastic Compute Cloud)

**Instance Types:**
| Family | Purpose | Example | Use Case |
|--------|---------|---------|----------|
| t3/t3a | General (burstable) | t3.medium | Dev/test, small apps |
| m5/m6i | General purpose | m5.xlarge | Web servers, app servers |
| c5/c6i | Compute optimized | c5.2xlarge | Batch processing, CI/CD |
| r5/r6i | Memory optimized | r5.large | Databases, caching |
| g4/p4 | GPU | g4dn.xlarge | ML, video processing |

```bash
# Launch an EC2 instance
aws ec2 run-instances \
  --image-id ami-0123456789abcdef0 \
  --instance-type t3.medium \
  --key-name my-key-pair \
  --security-group-ids sg-xxx \
  --subnet-id subnet-xxx \
  --iam-instance-profile Name=EC2-S3-Profile \
  --user-data file://startup.sh \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=web-server-1}]'

# User data script example (startup.sh)
#!/bin/bash
yum update -y
yum install -y httpd
systemctl start httpd
systemctl enable httpd
echo "<h1>Hello from $(hostname)</h1>" > /var/www/html/index.html

# List instances
aws ec2 describe-instances --filters "Name=instance-state-name,Values=running" \
  --query "Reservations[].Instances[].[InstanceId,InstanceType,PublicIpAddress,Tags[?Key=='Name'].Value|[0]]" \
  --output table

# Stop/Start/Terminate
aws ec2 stop-instances --instance-ids i-xxx
aws ec2 start-instances --instance-ids i-xxx
aws ec2 terminate-instances --instance-ids i-xxx
```

#### Auto Scaling Groups

```bash
# Create a Launch Template
aws ec2 create-launch-template --launch-template-name web-lt \
  --launch-template-data '{
    "ImageId": "ami-0123456789abcdef0",
    "InstanceType": "t3.medium",
    "KeyName": "my-key",
    "SecurityGroupIds": ["sg-xxx"],
    "IamInstanceProfile": {"Name": "EC2-S3-Profile"},
    "UserData": "base64-encoded-script"
  }'

# Create Auto Scaling Group
aws autoscaling create-auto-scaling-group --auto-scaling-group-name web-asg \
  --launch-template LaunchTemplateName=web-lt,Version='$Latest' \
  --min-size 2 --max-size 10 --desired-capacity 3 \
  --vpc-zone-identifier "subnet-xxx,subnet-yyy" \
  --target-group-arns arn:aws:elasticloadbalancing:me-south-1:123:targetgroup/web-tg/xxx \
  --health-check-type ELB --health-check-grace-period 300

# Create scaling policy (Target Tracking)
aws autoscaling put-scaling-policy --auto-scaling-group-name web-asg \
  --policy-name cpu-target-tracking \
  --policy-type TargetTrackingScaling \
  --target-tracking-configuration '{
    "PredefinedMetricSpecification": {
      "PredefinedMetricType": "ASGAverageCPUUtilization"
    },
    "TargetValue": 70.0
  }'
```

#### ECS (Elastic Container Service)

```bash
# Create ECS Cluster
aws ecs create-cluster --cluster-name production-cluster \
  --capacity-providers FARGATE FARGATE_SPOT

# Register Task Definition
aws ecs register-task-definition --cli-input-json '{
  "family": "web-app",
  "networkMode": "awsvpc",
  "requiresCompatibilities": ["FARGATE"],
  "cpu": "256",
  "memory": "512",
  "executionRoleArn": "arn:aws:iam::123:role/ecsTaskExecutionRole",
  "containerDefinitions": [{
    "name": "web",
    "image": "123456789012.dkr.ecr.me-south-1.amazonaws.com/web-app:latest",
    "portMappings": [{"containerPort": 80, "protocol": "tcp"}],
    "logConfiguration": {
      "logDriver": "awslogs",
      "options": {
        "awslogs-group": "/ecs/web-app",
        "awslogs-region": "me-south-1",
        "awslogs-stream-prefix": "ecs"
      }
    }
  }]
}'

# Create Service
aws ecs create-service --cluster production-cluster \
  --service-name web-service --task-definition web-app:1 \
  --desired-count 3 --launch-type FARGATE \
  --network-configuration '{
    "awsvpcConfiguration": {
      "subnets": ["subnet-xxx", "subnet-yyy"],
      "securityGroups": ["sg-xxx"],
      "assignPublicIp": "DISABLED"
    }
  }' \
  --load-balancers '[{
    "targetGroupArn": "arn:aws:elasticloadbalancing:...",
    "containerName": "web",
    "containerPort": 80
  }]'
```

**Fargate vs EC2 Launch Type:**
| Feature | Fargate | EC2 |
|---------|---------|-----|
| Management | Serverless, no EC2 to manage | You manage EC2 instances |
| Pricing | Per vCPU/memory per second | EC2 instance pricing |
| Scaling | Per task | Per instance + task |
| Best for | Variable workloads, small teams | Predictable, cost-sensitive |

#### EKS (Elastic Kubernetes Service)

```bash
# Create EKS cluster (using eksctl is easier)
eksctl create cluster --name production --region me-south-1 \
  --nodegroup-name workers --node-type m5.large --nodes 3 \
  --nodes-min 2 --nodes-max 5 --managed

# Update kubeconfig
aws eks update-kubeconfig --name production --region me-south-1

# List clusters
aws eks list-clusters
aws eks describe-cluster --name production
```

#### Lambda (Serverless)

```bash
# Create a Lambda function
aws lambda create-function --function-name process-order \
  --runtime python3.11 --handler lambda_function.handler \
  --role arn:aws:iam::123:role/LambdaExecutionRole \
  --zip-file fileb://function.zip \
  --timeout 30 --memory-size 256

# Invoke function
aws lambda invoke --function-name process-order \
  --payload '{"order_id": "12345"}' output.json

# Lambda Limits:
# - Timeout: max 15 minutes
# - Memory: 128 MB to 10 GB
# - Package size: 50 MB (zip), 250 MB (unzipped)
# - Concurrent executions: 1000 (default, can increase)
# - /tmp storage: 512 MB to 10 GB
```

---

### 2.4 Storage

#### S3 (Simple Storage Service)

```bash
# Create bucket
aws s3 mb s3://my-company-data-me-south-1 --region me-south-1

# Upload/download
aws s3 cp file.txt s3://my-bucket/path/
aws s3 cp s3://my-bucket/path/file.txt ./
aws s3 sync ./local-dir s3://my-bucket/backup/

# List objects
aws s3 ls s3://my-bucket/path/ --recursive

# Enable versioning
aws s3api put-bucket-versioning --bucket my-bucket \
  --versioning-configuration Status=Enabled

# Lifecycle policy
aws s3api put-bucket-lifecycle-configuration --bucket my-bucket \
  --lifecycle-configuration '{
    "Rules": [{
      "ID": "ArchiveOldData",
      "Status": "Enabled",
      "Filter": {"Prefix": "logs/"},
      "Transitions": [
        {"Days": 30, "StorageClass": "STANDARD_IA"},
        {"Days": 90, "StorageClass": "GLACIER"},
        {"Days": 365, "StorageClass": "DEEP_ARCHIVE"}
      ],
      "Expiration": {"Days": 730}
    }]
  }'
```

**S3 Storage Classes:**
| Class | Use Case | Retrieval | Cost |
|-------|----------|-----------|------|
| Standard | Frequently accessed | Instant | $$$ |
| Standard-IA | Infrequent access (30+ days) | Instant | $$ |
| One Zone-IA | Non-critical infrequent data | Instant | $ |
| Glacier Instant | Archive, instant access | Instant | $ |
| Glacier Flexible | Archive (minutes-hours) | 1-12 hours | ¢ |
| Glacier Deep Archive | Long-term archive | 12-48 hours | ¢¢ |

#### EBS (Elastic Block Store)

| Type | IOPS | Throughput | Use Case |
|------|------|-----------|----------|
| gp3 | 3000-16000 | 125-1000 MB/s | General purpose (default) |
| io2 | Up to 64000 | 1000 MB/s | Databases, critical apps |
| st1 | N/A | Up to 500 MB/s | Big data, log processing |
| sc1 | N/A | Up to 250 MB/s | Cold storage, infrequent |

```bash
# Create volume
aws ec2 create-volume --volume-type gp3 --size 100 \
  --availability-zone me-south-1a --iops 5000 --throughput 250

# Attach to instance
aws ec2 attach-volume --volume-id vol-xxx --instance-id i-xxx --device /dev/sdf

# Create snapshot
aws ec2 create-snapshot --volume-id vol-xxx --description "Daily backup"
```

#### EFS (Elastic File System)

```bash
# Create EFS filesystem
aws efs create-file-system --creation-token prod-efs \
  --performance-mode generalPurpose --throughput-mode bursting \
  --tags Key=Name,Value=shared-storage

# Create mount target (one per AZ)
aws efs create-mount-target --file-system-id fs-xxx \
  --subnet-id subnet-xxx --security-groups sg-xxx

# Mount on EC2 (after installing amazon-efs-utils)
sudo mount -t efs fs-xxx:/ /mnt/efs
```

---

### 2.5 Databases

#### RDS (Relational Database Service)

```bash
# Create RDS instance (Multi-AZ)
aws rds create-db-instance \
  --db-instance-identifier prod-db \
  --db-instance-class db.r5.large \
  --engine postgres --engine-version 15.4 \
  --master-username admin \
  --master-user-password 'SecureP@ss123!' \
  --allocated-storage 100 --storage-type gp3 \
  --multi-az \
  --db-subnet-group-name prod-db-subnet-group \
  --vpc-security-group-ids sg-xxx \
  --backup-retention-period 7 \
  --storage-encrypted \
  --kms-key-id arn:aws:kms:me-south-1:123:key/xxx

# Create read replica
aws rds create-db-instance-read-replica \
  --db-instance-identifier prod-db-reader \
  --source-db-instance-identifier prod-db \
  --db-instance-class db.r5.large

# Supported engines: PostgreSQL, MySQL, MariaDB, Oracle, SQL Server, Aurora
```

**Multi-AZ vs Read Replicas:**
- **Multi-AZ:** High availability, synchronous replication, automatic failover
- **Read Replicas:** Performance, asynchronous replication, manual promotion

#### Aurora

- MySQL and PostgreSQL compatible
- 5x performance of MySQL, 3x of PostgreSQL
- Auto-scales storage (10 GB to 128 TB)
- 6 copies of data across 3 AZs
- Aurora Serverless for variable workloads

#### DynamoDB

```bash
# Create table
aws dynamodb create-table --table-name Orders \
  --attribute-definitions \
    AttributeName=OrderId,AttributeType=S \
    AttributeName=CustomerId,AttributeType=S \
  --key-schema \
    AttributeName=OrderId,KeyType=HASH \
    AttributeName=CustomerId,KeyType=RANGE \
  --billing-mode PAY_PER_REQUEST

# Put item
aws dynamodb put-item --table-name Orders --item '{
  "OrderId": {"S": "ORD-001"},
  "CustomerId": {"S": "CUST-123"},
  "Amount": {"N": "99.99"},
  "Status": {"S": "Pending"}
}'

# Query
aws dynamodb query --table-name Orders \
  --key-condition-expression "OrderId = :id" \
  --expression-attribute-values '{":id": {"S": "ORD-001"}}'
```

**Capacity Modes:**
- **On-Demand:** Pay per request, auto-scales (good for unpredictable)
- **Provisioned:** Set RCU/WCU, cheaper for predictable workloads

#### ElastiCache

- **Redis:** Complex data structures, pub/sub, persistence, clustering
- **Memcached:** Simple key-value, multi-threaded, no persistence

```bash
# Create Redis cluster
aws elasticache create-replication-group \
  --replication-group-id prod-redis \
  --replication-group-description "Production cache" \
  --engine redis --cache-node-type cache.r6g.large \
  --num-cache-clusters 2
```

---

### 2.6 Load Balancing

#### ALB (Application Load Balancer - Layer 7)

```bash
# Create ALB
aws elbv2 create-load-balancer --name web-alb \
  --subnets subnet-xxx subnet-yyy \
  --security-groups sg-xxx --scheme internet-facing \
  --type application

# Create target group
aws elbv2 create-target-group --name web-targets \
  --protocol HTTP --port 80 --vpc-id vpc-xxx \
  --health-check-path /health --health-check-interval-seconds 30 \
  --healthy-threshold-count 2 --unhealthy-threshold-count 3 \
  --target-type instance

# Register targets
aws elbv2 register-targets --target-group-arn arn:xxx \
  --targets Id=i-xxx,Port=80 Id=i-yyy,Port=80

# Create listener
aws elbv2 create-listener --load-balancer-arn arn:xxx \
  --protocol HTTPS --port 443 \
  --certificates CertificateArn=arn:aws:acm:... \
  --default-actions Type=forward,TargetGroupArn=arn:xxx

# Path-based routing rule
aws elbv2 create-rule --listener-arn arn:xxx --priority 10 \
  --conditions '[{"Field":"path-pattern","Values":["/api/*"]}]' \
  --actions '[{"Type":"forward","TargetGroupArn":"arn:api-tg"}]'
```

**ALB Features:** Path-based routing, host-based routing, WebSocket, HTTP/2, gRPC, WAF integration

#### NLB (Network Load Balancer - Layer 4)

- Ultra-low latency, millions of requests per second
- Static IP per AZ, preserves source IP
- TCP, UDP, TLS protocols
- Best for: non-HTTP traffic, extreme performance, static IPs needed

---

### 2.7 Other Essential Services

#### Route 53 (DNS)

```bash
# Create hosted zone
aws route53 create-hosted-zone --name example.com --caller-reference $(date +%s)

# Create A record
aws route53 change-resource-record-sets --hosted-zone-id Z123 \
  --change-batch '{
    "Changes": [{
      "Action": "UPSERT",
      "ResourceRecordSet": {
        "Name": "api.example.com",
        "Type": "A",
        "AliasTarget": {
          "HostedZoneId": "Z35SXDOTRQ7X7K",
          "DNSName": "web-alb-xxx.me-south-1.elb.amazonaws.com",
          "EvaluateTargetHealth": true
        }
      }
    }]
  }'
```

**Routing Policies:** Simple, Weighted, Latency, Failover, Geolocation, Multi-value

#### CloudFront (CDN)

- Global content delivery network (400+ edge locations)
- Caches content close to users
- HTTPS, custom domains, Lambda@Edge
- Great for static websites, APIs, video streaming

#### SQS, SNS, EventBridge

```bash
# SQS — Message Queue
aws sqs create-queue --queue-name order-processing \
  --attributes VisibilityTimeout=60,MessageRetentionPeriod=86400

aws sqs send-message --queue-url https://sqs.me-south-1.amazonaws.com/123/order-processing \
  --message-body '{"order_id":"123","action":"process"}'

# SNS — Pub/Sub Notifications
aws sns create-topic --name deployment-alerts
aws sns subscribe --topic-arn arn:aws:sns:me-south-1:123:deployment-alerts \
  --protocol email --notification-endpoint team@company.com

# EventBridge — Event-driven architecture
aws events put-rule --name ec2-state-change \
  --event-pattern '{"source":["aws.ec2"],"detail-type":["EC2 Instance State-change Notification"]}'
```

#### ECR (Elastic Container Registry)

```bash
# Create repository
aws ecr create-repository --repository-name web-app

# Login to ECR
aws ecr get-login-password --region me-south-1 | \
  docker login --username AWS --password-stdin 123456789012.dkr.ecr.me-south-1.amazonaws.com

# Push image
docker tag web-app:latest 123456789012.dkr.ecr.me-south-1.amazonaws.com/web-app:latest
docker push 123456789012.dkr.ecr.me-south-1.amazonaws.com/web-app:latest
```

#### Secrets Manager & Parameter Store

```bash
# Secrets Manager (auto-rotation, $$)
aws secretsmanager create-secret --name prod/database/password \
  --secret-string '{"username":"admin","password":"SecureP@ss!"}'

aws secretsmanager get-secret-value --secret-id prod/database/password

# Parameter Store (simpler, free tier available)
aws ssm put-parameter --name "/prod/app/db-host" \
  --value "prod-db.xxx.me-south-1.rds.amazonaws.com" --type String

aws ssm put-parameter --name "/prod/app/api-key" \
  --value "secret-key-value" --type SecureString

aws ssm get-parameter --name "/prod/app/db-host"
aws ssm get-parameters-by-path --path "/prod/app/" --recursive
```

#### CloudWatch & CloudTrail

```bash
# CloudWatch — Metrics, Logs, Alarms
aws cloudwatch put-metric-alarm --alarm-name high-cpu \
  --metric-name CPUUtilization --namespace AWS/EC2 \
  --statistic Average --period 300 --threshold 80 \
  --comparison-operator GreaterThanThreshold \
  --evaluation-periods 2 --alarm-actions arn:aws:sns:...

# View logs
aws logs describe-log-groups
aws logs get-log-events --log-group-name /ecs/web-app \
  --log-stream-name ecs/web/xxx

# CloudTrail — API audit logging
aws cloudtrail describe-trails
aws cloudtrail lookup-events --lookup-attributes \
  AttributeKey=EventName,AttributeValue=RunInstances
```

#### Systems Manager (SSM)

```bash
# Run command on instances (no SSH needed!)
aws ssm send-command --instance-ids i-xxx \
  --document-name "AWS-RunShellScript" \
  --parameters 'commands=["uptime","df -h","free -m"]'

# Start session (SSH alternative)
aws ssm start-session --target i-xxx

# Patch management
aws ssm create-patch-baseline --name "ProdLinuxBaseline" \
  --operating-system AMAZON_LINUX_2 \
  --approval-rules '{...}'
```

---

### 2.8 AWS CLI Essentials

```bash
# Most commonly used AWS CLI commands for DevOps:

# Identity
aws sts get-caller-identity

# EC2
aws ec2 describe-instances --output table
aws ec2 describe-instances --filters "Name=tag:Environment,Values=production"

# S3
aws s3 ls
aws s3 sync . s3://bucket/path --exclude "*.tmp"
aws s3 presign s3://bucket/file.pdf --expires-in 3600

# ECS
aws ecs list-services --cluster production
aws ecs update-service --cluster prod --service web --force-new-deployment

# Logs
aws logs tail /ecs/web-app --follow --since 1h

# CloudFormation
aws cloudformation deploy --template-file template.yml \
  --stack-name my-stack --capabilities CAPABILITY_IAM

# Cost
aws ce get-cost-and-usage --time-period Start=2024-01-01,End=2024-01-31 \
  --granularity MONTHLY --metrics BlendedCost
```

---

## Section 3: Azure Cloud

### 3.1 Azure Fundamentals

#### Azure Hierarchy

```
┌────────────────────────────────────────────┐
│          Management Groups                   │
│  ┌──────────────────────────────────────┐  │
│  │  Root Management Group                │  │
│  │  ├── Production MG                    │  │
│  │  │   ├── Subscription: Prod-UAE       │  │
│  │  │   └── Subscription: Prod-KSA       │  │
│  │  ├── Development MG                   │  │
│  │  │   └── Subscription: Dev-UAE        │  │
│  │  └── Security MG                      │  │
│  │      └── Subscription: Security       │  │
│  └──────────────────────────────────────┘  │
│                                              │
│  Each Subscription contains:                 │
│  ├── Resource Group: rg-web-prod            │
│  │   ├── VM, Load Balancer, etc.            │
│  ├── Resource Group: rg-data-prod           │
│  │   ├── SQL Database, Storage, etc.        │
│  └── Resource Group: rg-network-prod        │
│      ├── VNet, NSG, etc.                    │
└────────────────────────────────────────────┘
```

**Key Concepts:**
- **Management Groups:** Organize subscriptions, apply policies at scale
- **Subscriptions:** Billing boundary, access control boundary
- **Resource Groups:** Logical container for resources (cannot be nested)
- **Resources:** Individual services (VMs, databases, etc.)

```bash
# az CLI login
az login
az login --service-principal -u <app-id> -p <password> --tenant <tenant-id>

# Account management
az account list --output table
az account set --subscription "Production-UAE"
az account show

# Resource groups
az group create --name rg-web-prod --location uaenorth
az group list --output table
az group delete --name rg-web-dev --yes --no-wait
```

#### Azure AD (Entra ID)

Microsoft Entra ID (formerly Azure Active Directory) is the identity platform:

```bash
# Users
az ad user create --display-name "DevOps Engineer" \
  --user-principal-name devops@company.onmicrosoft.com \
  --password "SecureP@ss123!"

az ad user list --query "[].{Name:displayName, UPN:userPrincipalName}" --output table

# Groups
az ad group create --display-name "DevOps-Team" --mail-nickname "devops-team"
az ad group member add --group "DevOps-Team" --member-id <user-object-id>

# Service Principals (application identity)
az ad sp create-for-rbac --name "github-actions-sp" \
  --role Contributor --scopes /subscriptions/<sub-id>/resourceGroups/rg-web-prod

# Managed Identities (preferred over service principals for Azure resources)
# System-assigned: tied to resource lifecycle
# User-assigned: independent lifecycle, can be shared
az identity create --name web-app-identity --resource-group rg-web-prod
```

**Service Principal vs Managed Identity:**
| Feature | Service Principal | Managed Identity |
|---------|-------------------|-----------------|
| Credentials | Password/certificate (you manage) | Azure manages automatically |
| Rotation | Manual | Automatic |
| Use case | External tools (GitHub, Jenkins) | Azure-to-Azure (VM→Key Vault) |
| Security | Risk of credential leak | No credentials to leak |

#### RBAC (Role-Based Access Control)

```bash
# Built-in roles
az role definition list --output table --query "[].{Name:roleName, Description:description}"

# Key built-in roles:
# Owner — Full access + assign roles
# Contributor — Full access, cannot assign roles
# Reader — View only
# User Access Administrator — Manage access only

# Assign role
az role assignment create --assignee devops@company.com \
  --role "Contributor" \
  --scope /subscriptions/<sub-id>/resourceGroups/rg-web-prod

# Custom role
az role definition create --role-definition '{
  "Name": "DevOps Operator",
  "Description": "Can manage VMs and containers but not networking",
  "Actions": [
    "Microsoft.Compute/*",
    "Microsoft.ContainerService/*",
    "Microsoft.ContainerRegistry/*"
  ],
  "NotActions": [],
  "AssignableScopes": ["/subscriptions/<sub-id>"]
}'

# List role assignments
az role assignment list --resource-group rg-web-prod --output table
```

#### az CLI Setup and Usage

```bash
# Install az CLI
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash

# Configure defaults
az configure --defaults location=uaenorth group=rg-web-prod

# Output formats: json (default), table, tsv, yaml
az vm list --output table

# Query with JMESPath
az vm list --query "[?powerState=='VM running'].{Name:name, Size:hardwareProfile.vmSize}" -o table

# Extensions
az extension add --name aks-preview
az extension list --output table

# Interactive mode
az interactive
```

---

### 3.2 Networking

#### VNet, Subnets, NSGs

```bash
# Create VNet
az network vnet create --name prod-vnet \
  --resource-group rg-network-prod \
  --address-prefix 10.0.0.0/16 \
  --location uaenorth

# Create subnets
az network vnet subnet create --name public-subnet \
  --vnet-name prod-vnet --resource-group rg-network-prod \
  --address-prefix 10.0.1.0/24

az network vnet subnet create --name app-subnet \
  --vnet-name prod-vnet --resource-group rg-network-prod \
  --address-prefix 10.0.10.0/24

az network vnet subnet create --name db-subnet \
  --vnet-name prod-vnet --resource-group rg-network-prod \
  --address-prefix 10.0.20.0/24

# Create NSG (Network Security Group)
az network nsg create --name web-nsg --resource-group rg-network-prod

# Add rules
az network nsg rule create --nsg-name web-nsg \
  --resource-group rg-network-prod \
  --name AllowHTTP --priority 100 \
  --direction Inbound --access Allow \
  --protocol Tcp --destination-port-ranges 80 443

az network nsg rule create --nsg-name web-nsg \
  --resource-group rg-network-prod \
  --name DenyAllInbound --priority 4096 \
  --direction Inbound --access Deny \
  --protocol '*' --destination-port-ranges '*'

# Associate NSG with subnet
az network vnet subnet update --name app-subnet \
  --vnet-name prod-vnet --resource-group rg-network-prod \
  --network-security-group web-nsg
```

#### Azure Firewall

```bash
# Create Azure Firewall (centralized network filtering)
az network firewall create --name prod-firewall \
  --resource-group rg-network-prod --location uaenorth

# Create firewall policy
az network firewall policy create --name prod-fw-policy \
  --resource-group rg-network-prod

# Application rule (FQDN-based)
az network firewall policy rule-collection-group create \
  --name DefaultRuleCollectionGroup \
  --policy-name prod-fw-policy \
  --resource-group rg-network-prod --priority 100

# Azure Firewall provides: Network rules, Application rules, DNAT rules, Threat intelligence
```

#### Application Gateway (L7 Load Balancer)

```bash
# Create Application Gateway with WAF
az network application-gateway create --name web-appgw \
  --resource-group rg-network-prod --location uaenorth \
  --sku WAF_v2 --capacity 2 \
  --vnet-name prod-vnet --subnet public-subnet \
  --http-settings-port 80 --http-settings-protocol Http \
  --frontend-port 443 --routing-rule-type Basic

# Features: SSL termination, URL-based routing, multi-site hosting, WAF, autoscaling
```

#### Azure Load Balancer (L4)

```bash
# Create public Load Balancer
az network lb create --name web-lb \
  --resource-group rg-network-prod \
  --sku Standard --frontend-ip-name lb-frontend \
  --backend-pool-name web-backend

# Create health probe
az network lb probe create --lb-name web-lb \
  --resource-group rg-network-prod \
  --name http-probe --protocol Http \
  --port 80 --path /health

# Create load balancing rule
az network lb rule create --lb-name web-lb \
  --resource-group rg-network-prod \
  --name http-rule --protocol Tcp \
  --frontend-port 80 --backend-port 80 \
  --frontend-ip-name lb-frontend \
  --backend-pool-name web-backend \
  --probe-name http-probe
```

#### VNet Peering

```bash
# Peer VNet-A to VNet-B (need both directions)
az network vnet peering create --name peer-to-vnetB \
  --resource-group rg-network-prod --vnet-name prod-vnet \
  --remote-vnet /subscriptions/<sub>/resourceGroups/rg-dev/providers/Microsoft.Network/virtualNetworks/dev-vnet \
  --allow-vnet-access true

# Peer VNet-B to VNet-A
az network vnet peering create --name peer-to-vnetA \
  --resource-group rg-dev --vnet-name dev-vnet \
  --remote-vnet /subscriptions/<sub>/resourceGroups/rg-network-prod/providers/Microsoft.Network/virtualNetworks/prod-vnet \
  --allow-vnet-access true
```

#### Private Endpoints

```bash
# Create Private Endpoint for Azure SQL
az network private-endpoint create --name sql-pe \
  --resource-group rg-network-prod \
  --vnet-name prod-vnet --subnet db-subnet \
  --private-connection-resource-id /subscriptions/<sub>/resourceGroups/rg-data/providers/Microsoft.Sql/servers/prod-sql \
  --group-id sqlServer --connection-name sql-pe-connection

# Private Link keeps traffic on Microsoft backbone, never traverses internet
```

#### Azure DNS

```bash
# Create DNS zone
az network dns zone create --name example.com --resource-group rg-network-prod

# Create A record
az network dns record-set a add-record --zone-name example.com \
  --resource-group rg-network-prod \
  --record-set-name www --ipv4-address 20.xxx.xxx.xxx

# Create CNAME
az network dns record-set cname set-record --zone-name example.com \
  --resource-group rg-network-prod \
  --record-set-name api --cname api-appgw.uaenorth.cloudapp.azure.com
```

#### Azure Network Architecture

```
┌─────────────────────────── VNet 10.0.0.0/16 ──────────────────────────────┐
│                                                                              │
│  ┌─ Public Subnet 10.0.1.0/24 ─────────────────────────────────────────┐  │
│  │  [Application Gateway / WAF]   [Azure Bastion]   [Azure Firewall]    │  │
│  └──────────────────────────────────────────────────────────────────────┘  │
│                                                                              │
│  ┌─ App Subnet 10.0.10.0/24 ───────────────────────────────────────────┐  │
│  │  [AKS Nodes]   [VM Scale Set]   [Container Instances]               │  │
│  └──────────────────────────────────────────────────────────────────────┘  │
│                                                                              │
│  ┌─ DB Subnet 10.0.20.0/24 ────────────────────────────────────────────┐  │
│  │  [Azure SQL - Private Endpoint]   [Redis Cache]   [Cosmos DB PE]     │  │
│  └──────────────────────────────────────────────────────────────────────┘  │
│                                                                              │
│  [NSGs on each subnet]    [Private Endpoints]    [Service Endpoints]       │
└──────────────────────────────────────────────────────────────────────────────┘
         │                              │
    [VNet Peering]              [Private Link to PaaS]
         │
    [Hub VNet with Firewall]
```

---

### 3.3 Compute

#### Virtual Machines & VM Scale Sets

```bash
# Create VM
az vm create --name web-vm-01 \
  --resource-group rg-web-prod \
  --image Ubuntu2204 \
  --size Standard_D2s_v3 \
  --admin-username azureuser \
  --ssh-key-values ~/.ssh/id_rsa.pub \
  --vnet-name prod-vnet --subnet app-subnet \
  --nsg web-nsg --public-ip-address "" \
  --custom-data cloud-init.yaml

# VM Scale Set (auto-scaling)
az vmss create --name web-vmss \
  --resource-group rg-web-prod \
  --image Ubuntu2204 \
  --instance-count 3 \
  --vm-sku Standard_D2s_v3 \
  --admin-username azureuser \
  --ssh-key-values ~/.ssh/id_rsa.pub \
  --vnet-name prod-vnet --subnet app-subnet \
  --lb web-lb --backend-pool-name web-backend \
  --upgrade-policy-mode Rolling

# Auto-scale rules
az monitor autoscale create --name web-autoscale \
  --resource-group rg-web-prod \
  --resource /subscriptions/<sub>/resourceGroups/rg-web-prod/providers/Microsoft.Compute/virtualMachineScaleSets/web-vmss \
  --min-count 2 --max-count 10 --count 3

az monitor autoscale rule create --autoscale-name web-autoscale \
  --resource-group rg-web-prod \
  --condition "Percentage CPU > 70 avg 5m" \
  --scale out 2

az monitor autoscale rule create --autoscale-name web-autoscale \
  --resource-group rg-web-prod \
  --condition "Percentage CPU < 30 avg 5m" \
  --scale in 1
```

#### AKS (Azure Kubernetes Service) — Detailed

```bash
# Create AKS cluster
az aks create --name prod-aks \
  --resource-group rg-web-prod \
  --node-count 3 \
  --node-vm-size Standard_D4s_v3 \
  --network-plugin azure \
  --vnet-subnet-id /subscriptions/<sub>/resourceGroups/rg-network-prod/providers/Microsoft.Network/virtualNetworks/prod-vnet/subnets/app-subnet \
  --service-cidr 10.1.0.0/16 \
  --dns-service-ip 10.1.0.10 \
  --enable-managed-identity \
  --enable-addons monitoring \
  --generate-ssh-keys

# Get credentials
az aks get-credentials --name prod-aks --resource-group rg-web-prod

# Scale node pool
az aks nodepool scale --name nodepool1 --cluster-name prod-aks \
  --resource-group rg-web-prod --node-count 5

# Add node pool (e.g., for GPU workloads)
az aks nodepool add --name gpupool --cluster-name prod-aks \
  --resource-group rg-web-prod \
  --node-vm-size Standard_NC6s_v3 --node-count 1 \
  --labels workload=gpu

# Enable cluster autoscaler
az aks nodepool update --name nodepool1 --cluster-name prod-aks \
  --resource-group rg-web-prod \
  --enable-cluster-autoscaler --min-count 2 --max-count 10

# Upgrade cluster
az aks get-upgrades --name prod-aks --resource-group rg-web-prod --output table
az aks upgrade --name prod-aks --resource-group rg-web-prod --kubernetes-version 1.28.0

# AKS + ACR integration
az aks update --name prod-aks --resource-group rg-web-prod \
  --attach-acr prodacr
```

**AKS Best Practices:**
- Use Azure CNI for production (pods get VNet IPs)
- Enable managed identity (not service principal)
- Use multiple node pools for different workloads
- Enable Azure Policy for Kubernetes
- Use Private clusters for sensitive environments

#### Azure Container Instances (ACI)

```bash
# Quick container deployment (serverless containers)
az container create --name quick-task \
  --resource-group rg-web-prod \
  --image myacr.azurecr.io/batch-job:latest \
  --cpu 2 --memory 4 \
  --restart-policy Never \
  --registry-login-server myacr.azurecr.io \
  --registry-username myacr \
  --registry-password <password>

# Good for: batch jobs, CI/CD agents, testing, burst workloads
```

#### Azure App Service

```bash
# Create App Service Plan
az appservice plan create --name web-plan \
  --resource-group rg-web-prod \
  --sku P1V3 --is-linux

# Create Web App
az webapp create --name my-web-app-prod \
  --resource-group rg-web-prod \
  --plan web-plan --runtime "NODE:18-lts"

# Deploy from container
az webapp create --name my-api-prod \
  --resource-group rg-web-prod \
  --plan web-plan \
  --deployment-container-image-name myacr.azurecr.io/api:latest

# Configure app settings (environment variables)
az webapp config appsettings set --name my-web-app-prod \
  --resource-group rg-web-prod \
  --settings DB_HOST=prod-sql.database.windows.net NODE_ENV=production

# Deployment slots (blue-green)
az webapp deployment slot create --name my-web-app-prod \
  --resource-group rg-web-prod --slot staging
az webapp deployment slot swap --name my-web-app-prod \
  --resource-group rg-web-prod --slot staging
```

#### Azure Functions

```bash
# Create Function App
az functionapp create --name order-processor \
  --resource-group rg-web-prod \
  --consumption-plan-location uaenorth \
  --runtime python --runtime-version 3.11 \
  --storage-account prodstorage

# Triggers: HTTP, Timer, Blob, Queue, Event Grid, Cosmos DB, Service Bus
# Pricing: Consumption (pay per execution) or Premium (pre-warmed)
```

---

### 3.4 Storage

#### Blob Storage

```bash
# Create storage account
az storage account create --name prodstorageuae \
  --resource-group rg-data-prod \
  --location uaenorth \
  --sku Standard_ZRS \
  --kind StorageV2 \
  --access-tier Hot

# Create container
az storage container create --name application-data \
  --account-name prodstorageuae --auth-mode login

# Upload/download blobs
az storage blob upload --account-name prodstorageuae \
  --container-name application-data \
  --file ./backup.tar.gz --name backups/2024/backup.tar.gz

az storage blob download --account-name prodstorageuae \
  --container-name application-data \
  --name backups/2024/backup.tar.gz --file ./backup.tar.gz

# List blobs
az storage blob list --account-name prodstorageuae \
  --container-name application-data --output table

# Set lifecycle policy (tier and delete)
az storage account management-policy create --account-name prodstorageuae \
  --resource-group rg-data-prod --policy '{
    "rules": [{
      "name": "archiveOldLogs",
      "type": "Lifecycle",
      "definition": {
        "filters": {"blobTypes": ["blockBlob"], "prefixMatch": ["logs/"]},
        "actions": {
          "baseBlob": {
            "tierToCool": {"daysAfterModificationGreaterThan": 30},
            "tierToArchive": {"daysAfterModificationGreaterThan": 90},
            "delete": {"daysAfterModificationGreaterThan": 365}
          }
        }
      }
    }]
  }'
```

**Storage Tiers:**
| Tier | Access Cost | Storage Cost | Use Case |
|------|-------------|--------------|----------|
| Hot | Low | High | Frequently accessed data |
| Cool | Medium | Medium | 30+ days infrequent access |
| Cold | Higher | Lower | 90+ days infrequent access |
| Archive | Highest | Lowest | 180+ days, offline access |

**Redundancy Options:**
- LRS: 3 copies in single datacenter
- ZRS: 3 copies across availability zones (recommended for production)
- GRS: 6 copies, 2 regions
- RA-GRS: GRS + read access to secondary region

#### Azure Managed Disks

```bash
# Create managed disk
az disk create --name data-disk-01 \
  --resource-group rg-web-prod \
  --size-gb 256 --sku Premium_LRS

# Attach to VM
az vm disk attach --vm-name web-vm-01 \
  --resource-group rg-web-prod \
  --name data-disk-01

# Disk types:
# Premium SSD (P-series) — Production workloads, databases
# Standard SSD (E-series) — Web servers, dev/test
# Standard HDD (S-series) — Backup, non-critical
# Ultra Disk — SAP HANA, top-tier databases (up to 160,000 IOPS)
```

#### Azure Files

```bash
# Create file share
az storage share-rm create --storage-account prodstorageuae \
  --resource-group rg-data-prod \
  --name shared-config --quota 100

# Mount on Linux (SMB)
sudo mount -t cifs //prodstorageuae.file.core.windows.net/shared-config /mnt/azure \
  -o vers=3.0,username=prodstorageuae,password=<key>,dir_mode=0777,file_mode=0777

# Azure Files supports: SMB 3.0, NFS 4.1, REST API
# Use cases: Shared config, legacy app lift-and-shift, persistent storage for containers
```

---

### 3.5 Databases

#### Azure SQL

```bash
# Create SQL Server
az sql server create --name prod-sql-uae \
  --resource-group rg-data-prod \
  --location uaenorth \
  --admin-user sqladmin \
  --admin-password 'SecureP@ss123!'

# Create database
az sql db create --server prod-sql-uae \
  --resource-group rg-data-prod \
  --name app-database \
  --service-objective S2 \
  --zone-redundant true

# Configure firewall (or use Private Endpoint)
az sql server firewall-rule create --server prod-sql-uae \
  --resource-group rg-data-prod \
  --name AllowAzureServices \
  --start-ip-address 0.0.0.0 --end-ip-address 0.0.0.0

# Enable auditing
az sql server audit-policy update --server prod-sql-uae \
  --resource-group rg-data-prod --state Enabled \
  --storage-account prodstorageuae
```

#### Azure Database for PostgreSQL/MySQL

```bash
# Create PostgreSQL Flexible Server
az postgres flexible-server create \
  --name prod-postgres \
  --resource-group rg-data-prod \
  --location uaenorth \
  --admin-user pgadmin \
  --admin-password 'SecureP@ss123!' \
  --sku-name Standard_D2ds_v4 \
  --tier GeneralPurpose \
  --storage-size 128 \
  --version 15 \
  --high-availability ZoneRedundant

# Create MySQL Flexible Server
az mysql flexible-server create \
  --name prod-mysql \
  --resource-group rg-data-prod \
  --location uaenorth \
  --admin-user mysqladmin \
  --admin-password 'SecureP@ss123!' \
  --sku-name Standard_D2ds_v4 \
  --storage-size 128 --version 8.0.21
```

#### Cosmos DB

```bash
# Create Cosmos DB account (multi-model, global distribution)
az cosmosdb create --name prod-cosmos \
  --resource-group rg-data-prod \
  --locations regionName=uaenorth failoverPriority=0 \
  --locations regionName=uaecentral failoverPriority=1 \
  --default-consistency-level Session

# Create database and container
az cosmosdb sql database create --account-name prod-cosmos \
  --resource-group rg-data-prod --name app-db

az cosmosdb sql container create --account-name prod-cosmos \
  --resource-group rg-data-prod --database-name app-db \
  --name orders --partition-key-path "/customerId" \
  --throughput 400

# APIs: SQL (Core), MongoDB, Cassandra, Gremlin, Table
# Consistency levels: Strong, Bounded Staleness, Session, Consistent Prefix, Eventual
```

#### Azure Cache for Redis

```bash
# Create Redis Cache
az redis create --name prod-redis \
  --resource-group rg-data-prod \
  --location uaenorth \
  --sku Premium --vm-size P1 \
  --shard-count 2 \
  --minimum-tls-version 1.2

# Get connection info
az redis show --name prod-redis --resource-group rg-data-prod \
  --query "{Host:hostName, Port:sslPort}" --output table
az redis list-keys --name prod-redis --resource-group rg-data-prod
```

---

### 3.6 Other Essential Services

#### Azure Key Vault

```bash
# Create Key Vault
az keyvault create --name prod-kv-uae \
  --resource-group rg-security-prod \
  --location uaenorth \
  --enable-rbac-authorization true

# Store secrets
az keyvault secret set --vault-name prod-kv-uae \
  --name "db-connection-string" \
  --value "Server=prod-sql.database.windows.net;Database=app;..."

# Retrieve secret
az keyvault secret show --vault-name prod-kv-uae --name "db-connection-string" \
  --query value --output tsv

# Store certificate
az keyvault certificate import --vault-name prod-kv-uae \
  --name api-cert --file ./cert.pfx --password "certpass"

# Key Vault references in App Service
# @Microsoft.KeyVault(SecretUri=https://prod-kv-uae.vault.azure.net/secrets/db-connection-string)
```

#### Azure Monitor & Log Analytics

```bash
# Create Log Analytics workspace
az monitor log-analytics workspace create --workspace-name prod-logs \
  --resource-group rg-monitoring-prod --location uaenorth

# Create metric alert
az monitor metrics alert create --name high-cpu-alert \
  --resource-group rg-web-prod \
  --scopes /subscriptions/<sub>/resourceGroups/rg-web-prod/providers/Microsoft.Compute/virtualMachines/web-vm-01 \
  --condition "avg Percentage CPU > 80" \
  --window-size 5m --evaluation-frequency 1m \
  --action /subscriptions/<sub>/resourceGroups/rg-monitoring-prod/providers/Microsoft.Insights/actionGroups/ops-team

# Query logs with KQL (Kusto Query Language)
az monitor log-analytics query --workspace prod-logs-id \
  --analytics-query "ContainerLog | where LogEntry contains 'error' | summarize count() by bin(TimeGenerated, 1h)"

# Diagnostic settings (send metrics/logs to workspace)
az monitor diagnostic-settings create --name send-to-logs \
  --resource /subscriptions/<sub>/resourceGroups/rg-web-prod/providers/Microsoft.Compute/virtualMachines/web-vm-01 \
  --workspace prod-logs-id \
  --metrics '[{"category":"AllMetrics","enabled":true}]' \
  --logs '[{"category":"Administrative","enabled":true}]'
```

#### Azure Container Registry (ACR)

```bash
# Create ACR
az acr create --name prodacruae --resource-group rg-web-prod \
  --sku Premium --location uaenorth

# Login
az acr login --name prodacruae

# Build and push (ACR Tasks - build in cloud)
az acr build --registry prodacruae --image web-app:v1.0 .

# Push from local
docker tag web-app:latest prodacruae.azurecr.io/web-app:latest
docker push prodacruae.azurecr.io/web-app:latest

# List repositories
az acr repository list --name prodacruae --output table

# Enable geo-replication
az acr replication create --registry prodacruae --location uaecentral
```

#### Azure DevOps (Reference to Month 4)

- Azure Repos, Pipelines, Boards, Artifacts, Test Plans
- Tight integration with Azure services
- Commonly used in GCC enterprises
- See Month 4 material for CI/CD pipelines

#### Azure Policy

```bash
# List built-in policies
az policy definition list --query "[?policyType=='BuiltIn'].{Name:displayName}" --output table

# Assign policy (e.g., require tags)
az policy assignment create --name require-env-tag \
  --policy "/providers/Microsoft.Authorization/policyDefinitions/871b6d14-..." \
  --scope /subscriptions/<sub-id> \
  --params '{"tagName":{"value":"Environment"}}'

# Common policies for GCC:
# - Allowed locations (restrict to UAE/Bahrain)
# - Require encryption on storage
# - Require NSG on subnets
# - Restrict VM sizes
# - Require tags
```

#### Microsoft Defender for Cloud

```bash
# Enable Defender for Cloud
az security pricing create --name VirtualMachines --tier Standard
az security pricing create --name SqlServers --tier Standard
az security pricing create --name KubernetesService --tier Standard

# View secure score
az security secure-score-controls list --output table

# Features:
# - Security posture management (CSPM)
# - Secure score with recommendations
# - Regulatory compliance (PCI DSS, ISO 27001, NIST)
# - Threat protection for VMs, containers, databases
# - Just-in-time VM access
```

---

### 3.7 az CLI Essentials

```bash
# Most commonly used az CLI commands for DevOps:

# Login and account
az login
az account set --subscription "Production"
az account show

# Resource management
az group list --output table
az resource list --resource-group rg-web-prod --output table

# VMs
az vm list --output table
az vm start --name web-vm-01 --resource-group rg-web-prod
az vm stop --name web-vm-01 --resource-group rg-web-prod --no-wait
az vm run-command invoke --name web-vm-01 --resource-group rg-web-prod \
  --command-id RunShellScript --scripts "uptime && df -h"

# AKS
az aks list --output table
az aks get-credentials --name prod-aks --resource-group rg-web-prod
az aks nodepool list --cluster-name prod-aks --resource-group rg-web-prod -o table

# Storage
az storage blob list --account-name prodstorageuae --container-name backups -o table

# Networking
az network vnet list --output table
az network nsg rule list --nsg-name web-nsg --resource-group rg-network-prod -o table

# Monitoring
az monitor activity-log list --resource-group rg-web-prod --output table
az monitor metrics list --resource /subscriptions/<sub>/... --metric "Percentage CPU"

# Deployment (ARM/Bicep)
az deployment group create --resource-group rg-web-prod \
  --template-file main.bicep --parameters @parameters.json

# Cost
az consumption usage list --start-date 2024-01-01 --end-date 2024-01-31
```

---

## Section 4: Monitoring & Observability

### 4.1 Observability Concepts

#### Three Pillars of Observability

| Pillar | What | Example Tools |
|--------|------|---------------|
| **Metrics** | Numerical measurements over time | Prometheus, CloudWatch, Azure Monitor |
| **Logs** | Discrete events with context | ELK, Loki, CloudWatch Logs |
| **Traces** | Request flow across services | Jaeger, Tempo, X-Ray |

**Metrics:** "CPU is at 85%" → Tells you WHAT is happening
**Logs:** "Error: connection refused to db:5432" → Tells you WHY
**Traces:** "Request took 3.2s, 2.8s was in payment-service" → Tells you WHERE

#### Golden Signals (Google SRE Book)

| Signal | Definition | Example Metric |
|--------|-----------|---------------|
| **Latency** | Time to service a request | http_request_duration_seconds |
| **Traffic** | Demand on the system | http_requests_total per second |
| **Errors** | Rate of failed requests | http_requests_total{status=~"5.."} |
| **Saturation** | How "full" the service is | CPU %, memory %, queue depth |

#### RED Method (for microservices)

- **Rate** — Requests per second
- **Errors** — Number of failed requests per second
- **Duration** — Time each request takes (latency distribution)

```
# RED PromQL Examples:
Rate:     sum(rate(http_requests_total[5m]))
Errors:   sum(rate(http_requests_total{status=~"5.."}[5m]))
Duration: histogram_quantile(0.99, sum(rate(http_request_duration_seconds_bucket[5m])) by (le))
```

#### USE Method (for infrastructure resources)

- **Utilization** — Average time resource was busy (or % capacity used)
- **Saturation** — Extra work queued that can't be serviced
- **Errors** — Count of error events

Apply USE to: CPU, Memory, Disk I/O, Network I/O

#### SLI, SLO, SLA

| Term | Definition | Example |
|------|-----------|---------|
| **SLI** (Service Level Indicator) | Metric measuring service quality | 99.2% of requests < 200ms |
| **SLO** (Service Level Objective) | Target value for an SLI | 99.9% requests should be < 200ms |
| **SLA** (Service Level Agreement) | Contract with consequences | 99.9% uptime or customer gets credits |

**Relationship:** SLI measures → SLO targets → SLA promises to customers

**Example:**
```
SLI: Percentage of HTTP requests completing successfully in < 500ms
     Current: 99.7%

SLO: 99.9% of requests should complete in < 500ms (per 30-day window)
     Budget: 0.1% = ~43 minutes of errors per month

SLA: We guarantee 99.5% availability. Below that → 10% billing credit
```

#### Error Budgets

Error budget = 100% - SLO target
- If SLO = 99.9%, error budget = 0.1% = 43.2 minutes/month
- When budget is consumed → freeze changes, focus on reliability
- When budget is healthy → ship features faster

```
Monthly error budget (99.9% SLO):
- 30 days × 24h × 60m = 43,200 minutes total
- 0.1% of 43,200 = 43.2 minutes of allowed downtime
- If you've used 30 minutes → 13.2 minutes remaining → slow down deployments
```

---

### 4.2 Prometheus

#### Architecture

```
┌──────────────┐     ┌───────────────────┐     ┌──────────────┐
│  Targets     │────▶│   Prometheus       │────▶│  Grafana     │
│  (exporters) │     │  ┌─────────────┐  │     │  (dashboards)│
│              │     │  │   TSDB      │  │     └──────────────┘
│  /metrics    │     │  └─────────────┘  │
└──────────────┘     │  ┌─────────────┐  │     ┌──────────────┐
                     │  │  PromQL     │  │────▶│ Alertmanager │
┌──────────────┐     │  └─────────────┘  │     │  (routing)   │
│ Service      │     │  ┌─────────────┐  │     └──────┬───────┘
│ Discovery    │────▶│  │ Scrape      │  │            │
│ (K8s, file)  │     │  │ Engine      │  │     ┌──────▼───────┐
└──────────────┘     │  └─────────────┘  │     │ Slack/PD/    │
                     └───────────────────┘     │ Email/Teams  │
                                               └──────────────┘
```

**Key Concepts:**
- **Pull-based:** Prometheus scrapes targets (not push)
- **TSDB:** Time-series database optimized for metrics
- **PromQL:** Query language for metrics
- **Labels:** Key-value pairs for dimensional data model
- **Exporters:** Expose metrics in Prometheus format

#### Installation

```bash
# Docker Compose setup
cat > docker-compose-monitoring.yml << 'EOF'
version: '3.8'
services:
  prometheus:
    image: prom/prometheus:v2.48.0
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - ./rules:/etc/prometheus/rules
      - prometheus_data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--storage.tsdb.retention.time=30d'
      - '--web.enable-lifecycle'

  alertmanager:
    image: prom/alertmanager:v0.26.0
    ports:
      - "9093:9093"
    volumes:
      - ./alertmanager.yml:/etc/alertmanager/alertmanager.yml

  grafana:
    image: grafana/grafana:10.2.0
    ports:
      - "3000:3000"
    volumes:
      - grafana_data:/var/lib/grafana
      - ./grafana/provisioning:/etc/grafana/provisioning
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin

  node-exporter:
    image: prom/node-exporter:v1.7.0
    ports:
      - "9100:9100"
    volumes:
      - /proc:/host/proc:ro
      - /sys:/host/sys:ro
      - /:/rootfs:ro
    command:
      - '--path.procfs=/host/proc'
      - '--path.sysfs=/host/sys'
      - '--path.rootfs=/rootfs'

volumes:
  prometheus_data:
  grafana_data:
EOF
```

#### prometheus.yml (Configuration)

```yaml
# /etc/prometheus/prometheus.yml
global:
  scrape_interval: 15s          # How often to scrape targets
  evaluation_interval: 15s       # How often to evaluate rules
  scrape_timeout: 10s            # Timeout for scrape requests

# Alertmanager configuration
alerting:
  alertmanagers:
    - static_configs:
        - targets: ['alertmanager:9093']

# Rule files
rule_files:
  - "rules/*.yml"

# Scrape configurations
scrape_configs:
  # Prometheus self-monitoring
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  # Node Exporter (system metrics)
  - job_name: 'node-exporter'
    static_configs:
      - targets: ['node-exporter:9100']
    relabel_configs:
      - source_labels: [__address__]
        target_label: instance
        regex: '(.+):\d+'

  # Application metrics
  - job_name: 'web-application'
    metrics_path: /metrics
    scheme: http
    static_configs:
      - targets: ['app1:8080', 'app2:8080', 'app3:8080']
        labels:
          environment: production
          team: backend

  # Kubernetes service discovery
  - job_name: 'kubernetes-pods'
    kubernetes_sd_configs:
      - role: pod
    relabel_configs:
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
        action: keep
        regex: true
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_path]
        action: replace
        target_label: __metrics_path__
        regex: (.+)
      - source_labels: [__address__, __meta_kubernetes_pod_annotation_prometheus_io_port]
        action: replace
        regex: ([^:]+)(?::\d+)?;(\d+)
        replacement: $1:$2
        target_label: __address__
      - source_labels: [__meta_kubernetes_namespace]
        target_label: namespace
      - source_labels: [__meta_kubernetes_pod_name]
        target_label: pod

  # EC2 service discovery
  - job_name: 'ec2-instances'
    ec2_sd_configs:
      - region: me-south-1
        port: 9100
        filters:
          - name: tag:monitoring
            values: ['enabled']
    relabel_configs:
      - source_labels: [__meta_ec2_tag_Name]
        target_label: instance_name
      - source_labels: [__meta_ec2_instance_type]
        target_label: instance_type

  # File-based service discovery
  - job_name: 'file-sd'
    file_sd_configs:
      - files:
          - '/etc/prometheus/file_sd/*.json'
        refresh_interval: 30s
```

#### Metric Types

| Type | Description | Example | Use Case |
|------|-------------|---------|----------|
| **Counter** | Only goes up (resets on restart) | http_requests_total | Request count, errors |
| **Gauge** | Can go up and down | temperature_celsius | CPU %, memory, connections |
| **Histogram** | Samples in configurable buckets | http_request_duration_seconds | Latency percentiles |
| **Summary** | Similar to histogram, calculates quantiles | go_gc_duration_seconds | Pre-calculated quantiles |

#### PromQL Deep Dive

```promql
# === RATE AND INCREASE ===

# Request rate (per second) over last 5 minutes
rate(http_requests_total[5m])

# Request rate by status code
sum by (status_code) (rate(http_requests_total[5m]))

# Total increase in requests over 1 hour
increase(http_requests_total[1h])

# === AGGREGATIONS ===

# Total requests across all instances
sum(rate(http_requests_total[5m]))

# Average CPU by instance
avg by (instance) (rate(node_cpu_seconds_total{mode!="idle"}[5m])) * 100

# Max memory usage per pod
max by (pod) (container_memory_usage_bytes{namespace="production"})

# Top 5 pods by CPU
topk(5, sum by (pod) (rate(container_cpu_usage_seconds_total[5m])))

# === LATENCY PERCENTILES (Histogram) ===

# 99th percentile latency
histogram_quantile(0.99,
  sum by (le) (rate(http_request_duration_seconds_bucket[5m]))
)

# 95th percentile by service
histogram_quantile(0.95,
  sum by (le, service) (rate(http_request_duration_seconds_bucket[5m]))
)

# 50th percentile (median)
histogram_quantile(0.50,
  sum by (le) (rate(http_request_duration_seconds_bucket[5m]))
)

# === ERROR RATE ===

# Error rate percentage
sum(rate(http_requests_total{status=~"5.."}[5m]))
/
sum(rate(http_requests_total[5m])) * 100

# Availability (success rate)
1 - (
  sum(rate(http_requests_total{status=~"5.."}[5m]))
  /
  sum(rate(http_requests_total[5m]))
) 

# === CPU AND MEMORY ===

# CPU usage percentage per node
100 - (avg by (instance) (irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)

# Memory usage percentage
(1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100

# Disk usage percentage
(1 - (node_filesystem_avail_bytes{mountpoint="/"} / node_filesystem_size_bytes{mountpoint="/"})) * 100

# Network traffic (bytes/sec)
rate(node_network_receive_bytes_total{device="eth0"}[5m])
rate(node_network_transmit_bytes_total{device="eth0"}[5m])

# === KUBERNETES SPECIFIC ===

# Pod restart count
increase(kube_pod_container_status_restarts_total[1h]) > 3

# Pods not ready
kube_pod_status_ready{condition="false"} == 1

# Container CPU throttling
rate(container_cpu_cfs_throttled_seconds_total[5m]) > 0

# PVC usage
kubelet_volume_stats_used_bytes / kubelet_volume_stats_capacity_bytes * 100

# === CUSTOM APPLICATION METRICS ===

# Queue depth (gauge)
order_queue_depth{service="payment"}

# Business metric: orders per minute
rate(orders_processed_total[5m]) * 60

# Cache hit ratio
sum(rate(cache_hits_total[5m])) / sum(rate(cache_requests_total[5m])) * 100
```

#### Recording Rules

```yaml
# /etc/prometheus/rules/recording_rules.yml
groups:
  - name: request_rates
    interval: 30s
    rules:
      # Pre-compute request rate (expensive query made cheap)
      - record: job:http_requests_total:rate5m
        expr: sum by (job) (rate(http_requests_total[5m]))

      - record: job:http_request_duration_seconds:p99
        expr: histogram_quantile(0.99, sum by (le, job) (rate(http_request_duration_seconds_bucket[5m])))

      - record: instance:node_cpu_utilisation:rate5m
        expr: 100 - (avg by (instance) (irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)

      - record: instance:node_memory_utilisation:ratio
        expr: 1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)
```

#### Alerting Rules

```yaml
# /etc/prometheus/rules/alerting_rules.yml
groups:
  - name: infrastructure
    rules:
      - alert: HighCPUUsage
        expr: instance:node_cpu_utilisation:rate5m > 80
        for: 5m
        labels:
          severity: warning
          team: infrastructure
        annotations:
          summary: "High CPU usage on {{ $labels.instance }}"
          description: "CPU usage is {{ $value }}% (threshold: 80%)"
          runbook_url: "https://wiki.company.com/runbooks/high-cpu"

      - alert: DiskSpaceLow
        expr: (1 - node_filesystem_avail_bytes{mountpoint="/"} / node_filesystem_size_bytes{mountpoint="/"}) * 100 > 85
        for: 10m
        labels:
          severity: critical
          team: infrastructure
        annotations:
          summary: "Disk space low on {{ $labels.instance }}"
          description: "Disk usage is {{ $value }}%"

      - alert: InstanceDown
        expr: up == 0
        for: 2m
        labels:
          severity: critical
        annotations:
          summary: "Instance {{ $labels.instance }} is down"

  - name: application
    rules:
      - alert: HighErrorRate
        expr: |
          sum(rate(http_requests_total{status=~"5.."}[5m]))
          /
          sum(rate(http_requests_total[5m])) > 0.05
        for: 5m
        labels:
          severity: critical
          team: backend
        annotations:
          summary: "Error rate above 5%"
          description: "Current error rate: {{ $value | humanizePercentage }}"

      - alert: HighLatency
        expr: histogram_quantile(0.99, sum by (le) (rate(http_request_duration_seconds_bucket[5m]))) > 2
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "P99 latency above 2 seconds"

      - alert: PodCrashLooping
        expr: increase(kube_pod_container_status_restarts_total[1h]) > 5
        for: 5m
        labels:
          severity: critical
          team: platform
        annotations:
          summary: "Pod {{ $labels.pod }} is crash looping"
```

---

### 4.3 Grafana

#### Installation and Setup

```bash
# Install Grafana (Ubuntu/Debian)
sudo apt-get install -y apt-transport-https software-properties-common
sudo mkdir -p /etc/apt/keyrings/
wget -q -O - https://apt.grafana.com/gpg.key | gpg --dearmor | sudo tee /etc/apt/keyrings/grafana.gpg > /dev/null
echo "deb [signed-by=/etc/apt/keyrings/grafana.gpg] https://apt.grafana.com stable main" | sudo tee /etc/apt/sources.list.d/grafana.list
sudo apt-get update && sudo apt-get install grafana
sudo systemctl enable grafana-server && sudo systemctl start grafana-server

# Default: http://localhost:3000 (admin/admin)
```

#### Dashboard Creation & Panel Types

**Panel Types:**
| Panel | Use Case | Example |
|-------|----------|---------|
| Time Series | Metrics over time | CPU usage, request rate |
| Stat | Single value with trend | Current error rate |
| Gauge | Value against thresholds | Disk usage % |
| Table | Tabular data | Top 10 pods by memory |
| Logs | Log entries | Application errors |
| Heatmap | Distribution over time | Latency buckets |
| Bar Chart | Comparisons | Requests by endpoint |

#### Variables and Templates

```json
// Dashboard variable definition (in dashboard JSON)
{
  "templating": {
    "list": [
      {
        "name": "namespace",
        "type": "query",
        "query": "label_values(kube_pod_info, namespace)",
        "refresh": 2
      },
      {
        "name": "pod",
        "type": "query",
        "query": "label_values(kube_pod_info{namespace=\"$namespace\"}, pod)",
        "refresh": 2
      },
      {
        "name": "interval",
        "type": "interval",
        "options": [{"value": "1m"}, {"value": "5m"}, {"value": "15m"}, {"value": "1h"}]
      }
    ]
  }
}
```

Usage in queries: `rate(http_requests_total{namespace="$namespace", pod="$pod"}[$interval])`

#### Dashboard Best Practices

1. **Top row:** Overview stats (request rate, error %, p99 latency, uptime)
2. **Second row:** Time series for golden signals
3. **Third row:** Resource utilization (CPU, memory, disk, network)
4. **Bottom:** Logs panel for correlated errors
5. Use variables for filtering (namespace, environment, service)
6. Set meaningful thresholds with color coding
7. Add links between dashboards for drill-down
8. Include annotations for deployments and incidents

#### Provisioning Dashboards as Code

```yaml
# /etc/grafana/provisioning/datasources/datasources.yml
apiVersion: 1
datasources:
  - name: Prometheus
    type: prometheus
    access: proxy
    url: http://prometheus:9090
    isDefault: true
  - name: Loki
    type: loki
    access: proxy
    url: http://loki:3100

# /etc/grafana/provisioning/dashboards/dashboards.yml
apiVersion: 1
providers:
  - name: 'default'
    orgId: 1
    folder: 'Infrastructure'
    type: file
    options:
      path: /var/lib/grafana/dashboards
      foldersFromFilesStructure: true
```

#### Popular Community Dashboards (IDs for import)

| Dashboard | ID | Description |
|-----------|-----|-------------|
| Node Exporter Full | 1860 | Complete Linux system metrics |
| Kubernetes Cluster | 7249 | Cluster overview |
| Docker Dashboard | 893 | Container metrics |
| NGINX Ingress | 9614 | Ingress controller |
| PostgreSQL | 9628 | Database metrics |
| Redis | 11835 | Cache metrics |

---

### 4.4 Alertmanager

#### Configuration

```yaml
# /etc/alertmanager/alertmanager.yml
global:
  resolve_timeout: 5m
  smtp_from: 'alerts@company.com'
  smtp_smarthost: 'smtp.office365.com:587'
  smtp_auth_username: 'alerts@company.com'
  smtp_auth_password: 'password'
  smtp_require_tls: true

# Templates
templates:
  - '/etc/alertmanager/templates/*.tmpl'

# Route tree
route:
  receiver: 'default-receiver'
  group_by: ['alertname', 'namespace', 'service']
  group_wait: 30s            # Wait before sending first notification
  group_interval: 5m         # Wait before sending update
  repeat_interval: 4h        # Re-send if not resolved
  routes:
    # Critical alerts → PagerDuty immediately
    - match:
        severity: critical
      receiver: 'pagerduty-critical'
      group_wait: 10s
      repeat_interval: 1h

    # Warning alerts → Slack
    - match:
        severity: warning
      receiver: 'slack-warnings'
      repeat_interval: 4h

    # Infrastructure team
    - match:
        team: infrastructure
      receiver: 'slack-infra'

    # Business hours only
    - match:
        severity: info
      receiver: 'email-team'
      active_time_intervals:
        - business-hours

# Receivers
receivers:
  - name: 'default-receiver'
    slack_configs:
      - api_url: 'https://hooks.slack.com/services/xxx/yyy/zzz'
        channel: '#alerts-general'
        title: '{{ .GroupLabels.alertname }}'
        text: '{{ range .Alerts }}{{ .Annotations.summary }}{{ end }}'

  - name: 'pagerduty-critical'
    pagerduty_configs:
      - routing_key: 'your-pagerduty-integration-key'
        severity: critical

  - name: 'slack-warnings'
    slack_configs:
      - api_url: 'https://hooks.slack.com/services/xxx/yyy/zzz'
        channel: '#alerts-warning'
        send_resolved: true

  - name: 'slack-infra'
    slack_configs:
      - api_url: 'https://hooks.slack.com/services/xxx/yyy/zzz'
        channel: '#infra-alerts'

  - name: 'email-team'
    email_configs:
      - to: 'devops-team@company.com'
        send_resolved: true

  - name: 'teams-channel'
    webhook_configs:
      - url: 'https://company.webhook.office.com/webhookb2/xxx'
        send_resolved: true

# Inhibition rules (suppress alerts when parent is firing)
inhibit_rules:
  - source_match:
      severity: critical
    target_match:
      severity: warning
    equal: ['alertname', 'instance']

  - source_match:
      alertname: InstanceDown
    target_match:
      severity: warning
    equal: ['instance']

# Time intervals
time_intervals:
  - name: business-hours
    time_intervals:
      - weekdays: ['monday:friday']
        times:
          - start_time: '08:00'
            end_time: '18:00'
```

#### Alert Fatigue Best Practices

1. **Actionable alerts only** — Every alert should require human action
2. **Proper severity levels** — Critical = page someone, Warning = review next day
3. **Appropriate thresholds** — Avoid alerting on temporary spikes (use `for` duration)
4. **Group related alerts** — Don't send 50 individual pod alerts when a node is down
5. **Use inhibition** — Suppress child alerts when parent is firing
6. **Regular review** — Delete alerts no one acts on
7. **Silence during maintenance** — Don't alert during planned downtime
8. **Route to the right team** — Don't send DB alerts to frontend team
9. **Include runbook links** — Make alerts actionable
10. **Set repeat intervals wisely** — Don't spam the same alert every 5 minutes

---

### 4.5 Logging

#### ELK Stack (Elasticsearch, Logstash, Kibana)

```
┌─────────┐     ┌──────────┐     ┌──────────────┐     ┌────────┐
│  Apps   │────▶│ Logstash │────▶│Elasticsearch │────▶│ Kibana │
│  Logs   │     │(process) │     │  (store &    │     │(visual)│
└─────────┘     └──────────┘     │   search)    │     └────────┘
                                  └──────────────┘
```

**Logstash Pipeline:**
```ruby
# /etc/logstash/conf.d/pipeline.conf
input {
  beats {
    port => 5044
  }
  tcp {
    port => 5000
    codec => json
  }
}

filter {
  if [type] == "nginx" {
    grok {
      match => { "message" => "%{COMBINEDAPACHELOG}" }
    }
    date {
      match => [ "timestamp", "dd/MMM/yyyy:HH:mm:ss Z" ]
    }
    geoip {
      source => "clientip"
    }
  }

  if [level] == "ERROR" {
    mutate {
      add_tag => ["error"]
    }
  }
}

output {
  elasticsearch {
    hosts => ["elasticsearch:9200"]
    index => "logs-%{+YYYY.MM.dd}"
  }
}
```

#### EFK Stack (Elasticsearch, Fluentd, Kibana)

Fluentd replaces Logstash — lighter, more Kubernetes-native:

```yaml
# Fluentd ConfigMap for Kubernetes
apiVersion: v1
kind: ConfigMap
metadata:
  name: fluentd-config
  namespace: logging
data:
  fluent.conf: |
    <source>
      @type tail
      path /var/log/containers/*.log
      pos_file /var/log/fluentd-containers.log.pos
      tag kubernetes.*
      read_from_head true
      <parse>
        @type json
        time_key time
        time_format %Y-%m-%dT%H:%M:%S.%NZ
      </parse>
    </source>

    <filter kubernetes.**>
      @type kubernetes_metadata
    </filter>

    <match kubernetes.**>
      @type elasticsearch
      host elasticsearch.logging.svc
      port 9200
      logstash_format true
      logstash_prefix k8s-logs
      <buffer>
        @type file
        path /var/log/fluentd-buffers/kubernetes.buffer
        flush_mode interval
        flush_interval 5s
      </buffer>
    </match>
```

#### Grafana Loki (Log Aggregation)

Loki is like "Prometheus for logs" — indexes labels, not full text:

```yaml
# Loki configuration (loki-config.yml)
auth_enabled: false

server:
  http_listen_port: 3100

common:
  path_prefix: /loki
  storage:
    filesystem:
      chunks_directory: /loki/chunks
      rules_directory: /loki/rules
  replication_factor: 1

schema_config:
  configs:
    - from: 2020-10-24
      store: boltdb-shipper
      object_store: filesystem
      schema: v11
      index:
        prefix: index_
        period: 24h

# Promtail (log shipper for Loki)
# promtail-config.yml
server:
  http_listen_port: 9080

positions:
  filename: /tmp/positions.yaml

clients:
  - url: http://loki:3100/loki/api/v1/push

scrape_configs:
  - job_name: kubernetes
    kubernetes_sd_configs:
      - role: pod
    pipeline_stages:
      - docker: {}
      - json:
          expressions:
            level: level
            msg: msg
      - labels:
          level:
```

**LogQL (Loki query language):**
```logql
# All logs from production namespace
{namespace="production"}

# Filter by content
{namespace="production", app="web"} |= "error"

# Regex filter
{app="api"} |~ "status=(4|5)\\d{2}"

# Parse JSON and filter
{app="api"} | json | level="error" | line_format "{{.msg}}"

# Rate of error logs
rate({app="api"} |= "error" [5m])

# Top 10 log-producing pods
topk(10, sum by (pod) (rate({namespace="production"}[5m])))
```

#### Structured Logging (JSON)

```json
// Good: Structured log entry
{
  "timestamp": "2024-01-15T10:30:45.123Z",
  "level": "error",
  "service": "payment-service",
  "trace_id": "abc123def456",
  "span_id": "span789",
  "message": "Payment processing failed",
  "error": "connection timeout",
  "customer_id": "CUST-456",
  "order_id": "ORD-789",
  "amount": 99.99,
  "duration_ms": 5002
}

// Bad: Unstructured log
// 2024-01-15 10:30:45 ERROR Payment failed for customer CUST-456 order ORD-789 - timeout
```

**Benefits of Structured Logging:**
- Easy to parse and search
- Consistent format across services
- Machine-readable
- Better aggregation and filtering
- Correlation via trace IDs

#### Log Levels Best Practices

| Level | When to Use | Example |
|-------|-------------|---------|
| FATAL | App cannot continue | Database connection lost permanently |
| ERROR | Operation failed, needs attention | Payment API returned 500 |
| WARN | Unexpected but handled | Retrying failed request (attempt 2/3) |
| INFO | Normal significant events | Order processed, User logged in |
| DEBUG | Diagnostic info (dev only) | Request payload, SQL query |
| TRACE | Very detailed flow info | Entering/exiting functions |

**Production:** INFO and above | **Staging:** DEBUG and above | **Never:** TRACE in production

#### Centralized Logging Architecture for Kubernetes

```
┌──────────────────────────────────────────────────────────┐
│                    Kubernetes Cluster                       │
│                                                            │
│  ┌──────┐  ┌──────┐  ┌──────┐   (app logs to stdout)    │
│  │Pod 1 │  │Pod 2 │  │Pod 3 │                           │
│  └──┬───┘  └──┬───┘  └──┬───┘                           │
│     │         │         │                                 │
│  ┌──▼─────────▼─────────▼──┐   (DaemonSet on each node) │
│  │  Promtail / Fluentd      │                            │
│  │  (reads container logs)  │                            │
│  └──────────┬───────────────┘                            │
└─────────────┼────────────────────────────────────────────┘
              │
              ▼
┌─────────────────────────────┐
│  Loki / Elasticsearch       │  (log storage)
└──────────────┬──────────────┘
               │
               ▼
┌─────────────────────────────┐
│  Grafana / Kibana           │  (visualization & search)
└─────────────────────────────┘
```

---

### 4.6 Distributed Tracing

#### What is Distributed Tracing?

In microservices, a single request may traverse 5-10 services. Distributed tracing follows the request through each service, measuring time spent and identifying bottlenecks.

```
User Request → [API Gateway: 5ms] → [Auth Service: 50ms] → [Order Service: 200ms]
                                                               ├── [DB Query: 150ms]
                                                               └── [Payment Service: 2000ms] ← BOTTLENECK
                                                                    └── [External API: 1900ms]
```

**Key Concepts:**
- **Trace:** Full journey of a request through all services
- **Span:** Single operation within a trace (one service call)
- **Context Propagation:** Passing trace ID between services (via headers)
- **Trace ID:** Unique identifier for the entire request

#### OpenTelemetry

OpenTelemetry (OTel) is the standard for instrumentation — merges OpenTracing + OpenCensus:

```yaml
# OpenTelemetry Collector configuration
receivers:
  otlp:
    protocols:
      grpc:
        endpoint: 0.0.0.0:4317
      http:
        endpoint: 0.0.0.0:4318

processors:
  batch:
    timeout: 5s
    send_batch_size: 1000
  memory_limiter:
    check_interval: 1s
    limit_mib: 512

exporters:
  jaeger:
    endpoint: jaeger:14250
    tls:
      insecure: true
  prometheus:
    endpoint: 0.0.0.0:8889

service:
  pipelines:
    traces:
      receivers: [otlp]
      processors: [memory_limiter, batch]
      exporters: [jaeger]
    metrics:
      receivers: [otlp]
      processors: [memory_limiter, batch]
      exporters: [prometheus]
```

#### Jaeger

Open-source distributed tracing system:
```bash
# Deploy Jaeger (all-in-one for dev)
docker run -d --name jaeger \
  -p 16686:16686 \   # UI
  -p 14250:14250 \   # gRPC collector
  -p 14268:14268 \   # HTTP collector
  jaegertracing/all-in-one:1.52

# Production: use Jaeger Operator on Kubernetes
# with Elasticsearch or Cassandra backend
```

#### Grafana Tempo

Distributed tracing backend that integrates with Grafana:
- Cost-effective (stores traces in object storage like S3)
- Integrates natively with Grafana, Loki, Prometheus
- Search by trace ID, service name, duration
- Pairs with Loki for "logs to traces" correlation

#### Correlation IDs

```
# HTTP Headers for context propagation:
traceparent: 00-4bf92f3577b34da6a3ce929d0e0e4736-00f067aa0ba902b7-01
X-Request-ID: req-abc-123
X-Correlation-ID: corr-def-456

# In application code, include trace_id in every log:
logger.info("Processing order", extra={
    "trace_id": get_current_trace_id(),
    "order_id": order.id,
    "customer_id": order.customer_id
})
```

---

### 4.7 Kubernetes Monitoring

#### kube-prometheus-stack (Helm Chart)

The standard way to deploy monitoring on Kubernetes:

```bash
# Add Helm repo
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

# Install kube-prometheus-stack
helm install monitoring prometheus-community/kube-prometheus-stack \
  --namespace monitoring --create-namespace \
  --set prometheus.prometheusSpec.retention=30d \
  --set prometheus.prometheusSpec.storageSpec.volumeClaimTemplate.spec.resources.requests.storage=50Gi \
  --set grafana.adminPassword=SecureP@ss \
  --set alertmanager.alertmanagerSpec.storage.volumeClaimTemplate.spec.resources.requests.storage=10Gi

# What it installs:
# - Prometheus (metrics collection)
# - Grafana (dashboards)
# - Alertmanager (alert routing)
# - Node Exporter (host metrics)
# - kube-state-metrics (K8s object metrics)
# - Default dashboards and alert rules
```

#### Node Exporter

Exposes hardware and OS metrics:
```
# Key metrics from Node Exporter:
node_cpu_seconds_total          — CPU usage by mode
node_memory_MemTotal_bytes      — Total memory
node_memory_MemAvailable_bytes  — Available memory
node_filesystem_size_bytes      — Disk total
node_filesystem_avail_bytes     — Disk available
node_network_receive_bytes_total — Network in
node_network_transmit_bytes_total — Network out
node_load1, node_load5, node_load15 — System load
```

#### kube-state-metrics

Exposes Kubernetes object state:
```
# Key metrics:
kube_pod_status_phase                    — Pod phase (Running, Pending, Failed)
kube_pod_container_status_restarts_total — Container restarts
kube_deployment_status_replicas          — Current replicas
kube_deployment_spec_replicas            — Desired replicas
kube_node_status_condition               — Node conditions (Ready, DiskPressure)
kube_pod_container_resource_requests     — Resource requests
kube_pod_container_resource_limits       — Resource limits
kube_hpa_status_current_replicas         — HPA current state
```

#### Metrics Server (for HPA)

```bash
# Install metrics server (required for kubectl top and HPA)
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

# Usage
kubectl top nodes
kubectl top pods -n production
kubectl top pods --sort-by=memory

# HPA uses metrics-server for CPU/memory auto-scaling
kubectl autoscale deployment web-app --cpu-percent=70 --min=3 --max=10
```

#### Common Alert Rules for Kubernetes

```yaml
# Essential K8s alerts
groups:
  - name: kubernetes-alerts
    rules:
      - alert: KubeNodeNotReady
        expr: kube_node_status_condition{condition="Ready",status="true"} == 0
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "Node {{ $labels.node }} is not ready"

      - alert: KubePodCrashLooping
        expr: increase(kube_pod_container_status_restarts_total[1h]) > 5
        for: 5m
        labels:
          severity: critical

      - alert: KubeDeploymentReplicasMismatch
        expr: kube_deployment_status_replicas != kube_deployment_spec_replicas
        for: 10m
        labels:
          severity: warning
        annotations:
          summary: "Deployment {{ $labels.deployment }} has mismatched replicas"

      - alert: KubePersistentVolumeFillingUp
        expr: kubelet_volume_stats_used_bytes / kubelet_volume_stats_capacity_bytes > 0.85
        for: 5m
        labels:
          severity: warning

      - alert: KubeContainerOOMKilled
        expr: kube_pod_container_status_last_terminated_reason{reason="OOMKilled"} == 1
        for: 0m
        labels:
          severity: critical
        annotations:
          summary: "Container {{ $labels.container }} OOM killed in pod {{ $labels.pod }}"

      - alert: KubeHPAMaxedOut
        expr: kube_hpa_status_current_replicas == kube_hpa_spec_max_replicas
        for: 15m
        labels:
          severity: warning
        annotations:
          summary: "HPA {{ $labels.hpa }} is at maximum replicas"
```

---

### 4.8 Incident Management

#### On-Call Practices

1. **Rotation schedule** — Weekly rotation, 2 people minimum
2. **Escalation path** — Primary → Secondary → Team Lead → Manager
3. **Response time SLAs** — P1: 5 min, P2: 30 min, P3: 4 hours
4. **Compensation** — On-call allowance, time-off for incidents
5. **Handoff** — Document active issues at rotation change
6. **Tools** — PagerDuty, Opsgenie, VictorOps (Splunk On-Call)

#### Incident Response Process

```
1. DETECT    → Alert fires or customer reports issue
2. TRIAGE    → Assess severity (P1-P4), assign incident commander
3. RESPOND   → Assemble team, start war room, investigate
4. MITIGATE  → Apply quick fix (rollback, restart, scale up)
5. RESOLVE   → Implement permanent fix, verify
6. REVIEW    → Postmortem within 48 hours
```

**Severity Levels:**
| Level | Impact | Response | Example |
|-------|--------|----------|---------|
| P1 | Service down, all users affected | Immediate page, all hands | Payment system down |
| P2 | Major feature broken | Page on-call, 15 min response | Search not working |
| P3 | Minor feature broken | Next business day | Reports delayed |
| P4 | Cosmetic/minor | Sprint backlog | UI misalignment |

#### Runbooks

A runbook is a step-by-step procedure for responding to an alert:

```markdown
## Runbook: High CPU Alert

### Alert: HighCPUUsage (> 80% for 5 minutes)

### Quick Diagnosis
1. Check which process is consuming CPU:
   `kubectl top pods -n production --sort-by=cpu`
2. Check for recent deployments:
   `kubectl rollout history deployment/web-app -n production`
3. Check for traffic spike:
   Look at Grafana dashboard: "Request Rate" panel

### Common Causes
- Traffic spike → Scale up replicas
- Memory leak → Restart pods, investigate later
- Bad deployment → Rollback
- Cron job/batch process → Wait or kill

### Resolution Steps
**If traffic spike:**
```
kubectl scale deployment web-app -n production --replicas=10
```

**If bad deployment:**
```
kubectl rollout undo deployment/web-app -n production
```

**If memory leak:**
```
kubectl delete pod <pod-name> -n production
# (deployment controller will create new one)
```

### Escalation
If not resolved in 30 minutes → Escalate to Team Lead
```

#### Postmortems (Blameless)

**Template:**
```markdown
## Incident Postmortem: [Title]

**Date:** 2024-01-15
**Duration:** 2 hours 15 minutes (10:30 - 12:45 UTC)
**Severity:** P1
**Impact:** 100% of users unable to process payments
**Incident Commander:** Jane Doe

### Summary
Payment service became unresponsive due to database connection pool exhaustion.

### Timeline
- 10:30 — Alert: PaymentServiceHighLatency fires
- 10:35 — On-call acknowledges, begins investigation
- 10:45 — Identified DB connection pool at 100%
- 11:00 — Attempted restart of payment pods
- 11:15 — Root cause identified: connection leak in v2.3.1
- 11:30 — Rollback to v2.3.0 initiated
- 12:00 — Rollback complete, monitoring
- 12:45 — All metrics normal, incident resolved

### Root Cause
Code change in v2.3.1 introduced a code path where DB connections
were not properly returned to the pool on timeout errors.

### Action Items
| Action | Owner | Due Date | Status |
|--------|-------|----------|--------|
| Fix connection leak in code | Dev Team | 2024-01-17 | Done |
| Add connection pool metrics alert | SRE | 2024-01-18 | Done |
| Add integration test for DB timeouts | Dev Team | 2024-01-22 | In Progress |
| Review connection pool settings | DBA | 2024-01-19 | Done |

### Lessons Learned
- Connection pool exhaustion had no alert (now added)
- Rollback process took 30 min (target: 5 min) — automate
- No load testing on the changed code path

### What Went Well
- Alert fired within 5 minutes of impact
- Team assembled quickly
- Clear rollback procedure existed
```

#### Escalation Policies

```yaml
# PagerDuty-style escalation policy
escalation_policy:
  name: "Production Services"
  repeat_enabled: true
  num_loops: 3
  escalation_rules:
    - escalation_delay_in_minutes: 5
      targets:
        - type: user
          name: "Primary On-Call"
    - escalation_delay_in_minutes: 15
      targets:
        - type: user
          name: "Secondary On-Call"
        - type: schedule
          name: "Engineering Managers"
    - escalation_delay_in_minutes: 30
      targets:
        - type: user
          name: "VP Engineering"
```

---

## Section 5: Best Practices

### Cloud Architecture Best Practices

1. **Design for failure** — Assume any component can fail; use multi-AZ, health checks, auto-recovery
2. **Use Infrastructure as Code** — All resources defined in Terraform/Bicep/CloudFormation
3. **Implement least privilege** — Minimal permissions for every identity and service
4. **Enable encryption everywhere** — At rest (KMS/Key Vault) and in transit (TLS)
5. **Use managed services** — Let the cloud provider handle undifferentiated heavy lifting
6. **Implement proper tagging** — Environment, team, cost center, project on every resource
7. **Design for scalability** — Stateless applications, external session stores, horizontal scaling
8. **Separate environments** — Different accounts/subscriptions for prod, staging, dev
9. **Use private networking** — Private endpoints, no public IPs on databases/internal services
10. **Implement backup and DR** — Cross-region replication, tested recovery procedures

### Monitoring Best Practices

11. **Monitor the four golden signals** — Latency, traffic, errors, saturation for every service
12. **Alert on symptoms, not causes** — Alert on "users experiencing errors" not "CPU is high"
13. **Use dashboards with purpose** — Overview → drill-down hierarchy, not one massive dashboard
14. **Implement SLOs before alerting** — Define what "healthy" means, then alert on SLO breach
15. **Correlate metrics, logs, and traces** — Use trace IDs across all three pillars
16. **Monitor from the outside in** — Synthetic checks, endpoint monitoring, not just internal metrics
17. **Set up on-call rotation properly** — No single point of failure in incident response
18. **Document runbooks for every alert** — If it alerts, there must be a documented response
19. **Review and prune alerts quarterly** — Remove alerts that are never actionable

### Cost Optimization Best Practices

20. **Right-size regularly** — Review instance utilization monthly, downsize overprovisioned resources
21. **Use spot/preemptible for batch** — CI/CD runners, batch jobs, dev environments
22. **Implement auto-shutdown** — Dev/test environments off nights and weekends (save 65%)
23. **Reserved instances for stable workloads** — 1-3 year commitments for production baseline
24. **Monitor unused resources** — Unattached disks, idle load balancers, unused elastic IPs
25. **Use storage lifecycle policies** — Automatically tier and delete old data

---

## Section 6: Lessons Learned / Pro Tips

1. **Start with monitoring** — Set up monitoring before deploying applications; you can't fix what you can't see

2. **Azure dominates GCC enterprise** — Most government and large enterprise contracts in UAE/KSA/Qatar use Azure. Learn it deeply for GCC jobs

3. **Always use managed identities over service principals** — In Azure, managed identities eliminate credential management headaches

4. **Instance profiles are your friend** — Never hardcode AWS credentials on EC2. Use instance profiles with IAM roles

5. **Tag everything from day one** — Retroactively tagging resources is painful. Enforce tagging with policies

6. **Prometheus is pull-based for a reason** — It simplifies discovery and avoids push storms. Don't fight it

7. **PromQL rate() needs at least 2 data points** — Use a range that covers at least 4x your scrape interval (15s scrape → [1m] minimum)

8. **Loki is NOT Elasticsearch** — Loki indexes labels only, not full text. Design your label strategy carefully

9. **Use recording rules for dashboard queries** — Pre-compute expensive queries. Dashboards load faster, and you can alert on recorded metrics

10. **Never alert on raw metrics** — Always use `for` duration. A 1-second CPU spike at 95% is not an incident

11. **VPC CIDR planning matters** — You cannot change VPC CIDR easily. Plan for growth. Use /16 for production VPCs

12. **NAT Gateway is expensive** — In AWS, NAT Gateway charges per hour AND per GB. Consider VPC endpoints for S3/DynamoDB

13. **Private endpoints save money AND improve security** — Traffic stays on provider backbone, no data transfer charges

14. **Use SSM Session Manager instead of bastion hosts** — No SSH keys to manage, full audit trail, no public IPs needed

15. **Test your disaster recovery** — An untested backup is not a backup. Run DR drills quarterly

16. **Multi-AZ ≠ Multi-Region** — Multi-AZ handles datacenter failures. Multi-region handles regional disasters. Know which you need

17. **Cost alerts should be set at 50%, 80%, and 100%** — Don't wait until you've blown the budget

18. **Use separate AWS accounts/Azure subscriptions per environment** — Blast radius isolation is critical

19. **Kubernetes monitoring needs all three: node-exporter + kube-state-metrics + cAdvisor** — Each provides different dimensions

20. **Structured logging pays off exponentially** — Invest time in JSON logging early. It makes debugging production 10x easier

21. **Always encrypt at rest and in transit** — This is non-negotiable in GCC markets with strict compliance requirements

22. **Data residency is a real constraint** — Many GCC clients require data to stay within specific countries. Verify region availability before architecture decisions

23. **Error budgets change the conversation** — Instead of "zero bugs" (impossible), discuss "acceptable error rate" — it aligns dev and ops

24. **Grafana dashboards as code** — Version control your dashboards. Manual dashboard creation doesn't survive team changes

25. **Learn KQL for Azure** — Kusto Query Language is essential for Azure Monitor/Log Analytics. It's different from PromQL

---

## Section 7: Interview Questions

### AWS Specific Questions (10)

**Q1: What is the difference between Security Groups and NACLs?**
> Security Groups are stateful (return traffic auto-allowed), operate at instance level, and only support allow rules. NACLs are stateless (must explicitly allow return traffic), operate at subnet level, support allow AND deny rules, and are processed in order by rule number.

**Q2: Explain the difference between an IAM Role and an IAM User.**
> A User has permanent credentials (password, access keys) tied to a person. A Role has no permanent credentials — it provides temporary security credentials via STS AssumeRole. Roles are used for cross-account access, service-to-service communication (EC2 to S3), and federated access.

**Q3: How does Auto Scaling work with target tracking?**
> Target tracking maintains a specific metric at a target value. Example: keep average CPU at 70%. ASG automatically adds instances when metric exceeds target and removes when it drops below. It handles both scale-out and scale-in with configurable cooldown periods.

**Q4: What are VPC Endpoints and why use them?**
> VPC Endpoints allow private connectivity to AWS services without traversing the internet. Gateway endpoints (S3, DynamoDB) are free and added to route tables. Interface endpoints (most other services) create an ENI in your subnet. Benefits: improved security, lower latency, reduced NAT Gateway costs.

**Q5: Explain S3 storage classes and when to use each.**
> Standard (frequent access), Standard-IA (30+ days, instant retrieval), One Zone-IA (non-critical infrequent), Glacier Instant (archive, instant), Glacier Flexible (1-12 hours), Deep Archive (12-48 hours). Use lifecycle policies to automatically transition objects based on age.

**Q6: What is the difference between ECS Fargate and ECS EC2 launch type?**
> Fargate is serverless — you define CPU/memory per task and AWS manages the infrastructure. EC2 launch type requires you to provision and manage EC2 instances. Fargate is simpler but more expensive per unit; EC2 gives more control and can be cheaper at scale.

**Q7: How do you implement cross-account access in AWS?**
> Account B creates a role with a trust policy allowing Account A. A user/role in Account A calls sts:AssumeRole to get temporary credentials for Account B. This avoids sharing long-term credentials and provides an audit trail via CloudTrail.

**Q8: What is AWS Organizations and why use it?**
> Organizations enables central management of multiple AWS accounts. Benefits: consolidated billing, Service Control Policies (guardrails), automated account provisioning, and organizational unit structure. Best practice: separate accounts for production, dev, security, and shared services.

**Q9: Explain Multi-AZ vs Read Replicas in RDS.**
> Multi-AZ: synchronous replication to standby in different AZ, automatic failover, for high availability. Read Replicas: asynchronous replication, can be in different regions, for read scaling. Multi-AZ standby cannot serve read traffic; read replicas can be promoted to standalone.

**Q10: What is the Shared Responsibility Model?**
> AWS is responsible for security OF the cloud (physical infrastructure, hypervisor, network). Customer is responsible for security IN the cloud (data, IAM, OS patching for EC2, network configuration, encryption). The division shifts based on service type (IaaS vs PaaS vs SaaS).

---

### Azure Specific Questions (10)

**Q1: Explain the Azure resource hierarchy.**
> Management Groups → Subscriptions → Resource Groups → Resources. Management Groups organize subscriptions and apply policies. Subscriptions are billing and access boundaries. Resource Groups are logical containers (cannot be nested) that share lifecycle and region.

**Q2: What is the difference between a Service Principal and a Managed Identity?**
> Service Principals have credentials you must manage (passwords/certificates that expire). Managed Identities have credentials managed automatically by Azure — no passwords to rotate, no secrets to leak. Use managed identities for Azure-to-Azure communication, service principals for external tools.

**Q3: How does Azure RBAC work?**
> RBAC assigns roles at a scope (management group, subscription, resource group, or resource). A role assignment = security principal + role definition + scope. Roles inherit downward. Key built-in roles: Owner, Contributor, Reader. Custom roles can define specific action sets.

**Q4: What is the difference between NSG and Azure Firewall?**
> NSG operates at subnet/NIC level with basic L3/L4 rules (like AWS Security Groups). Azure Firewall is a managed, centralized firewall with L7 capabilities, FQDN filtering, threat intelligence, and logging. Use NSGs for micro-segmentation, Azure Firewall for centralized network security.

**Q5: Explain AKS networking options (kubenet vs Azure CNI).**
> Kubenet: pods get IPs from a separate network, NAT to VNet. Simple but limited. Azure CNI: every pod gets a VNet IP directly, enabling direct connectivity with Azure services and VNet resources. Use CNI for production — it's required for some features (Windows nodes, Azure Network Policies).

**Q6: What are Azure Private Endpoints?**
> Private Endpoints assign a private IP from your VNet to a PaaS service (SQL, Storage, Key Vault). Traffic stays on Microsoft backbone, never traverses public internet. Eliminates need for service endpoints or firewall rules. Essential for security-sensitive GCC deployments.

**Q7: How does Azure Policy differ from RBAC?**
> RBAC controls WHO can do things (user permissions). Azure Policy controls WHAT can be done (resource configuration). Example: RBAC allows a user to create VMs. Policy restricts them to specific VM sizes or requires certain tags. Policies apply regardless of who performs the action.

**Q8: Explain Azure Monitor and Log Analytics.**
> Azure Monitor collects metrics and logs from all Azure resources. Log Analytics is the workspace where logs are stored and queried using KQL (Kusto Query Language). Together they provide alerting, dashboards, workbooks, and integration with external tools. Think of it as Azure's built-in Prometheus + Grafana.

**Q9: What is Cosmos DB and when would you use it?**
> Cosmos DB is a globally distributed, multi-model database. Use it for: global applications needing low-latency reads worldwide, applications with flexible schema needs, and when you need multiple data models (document, graph, key-value). It offers 5 consistency levels from strong to eventual.

**Q10: How do you implement blue-green deployments in Azure?**
> Azure App Service: use deployment slots (staging slot → swap to production). AKS: use service mesh or Kubernetes deployments with service switching. VM Scale Sets: create new VMSS, test, swap traffic via load balancer. Azure DevOps pipelines can orchestrate all approaches.

---

### Monitoring Specific Questions (10)

**Q1: What are the three pillars of observability?**
> Metrics (numerical measurements over time — e.g., CPU 85%), Logs (discrete events with context — e.g., error messages), and Traces (request flow across distributed services — e.g., where time is spent). Together they answer: what's happening, why, and where.

**Q2: Explain Prometheus architecture and how scraping works.**
> Prometheus uses a pull model — it periodically scrapes HTTP endpoints (/metrics) on configured targets. Architecture: scrape engine → TSDB (time-series database) → PromQL engine → API. It supports service discovery (Kubernetes, EC2, file-based) to find targets dynamically. Alertmanager handles alert routing.

**Q3: What is the difference between Counter and Gauge metric types?**
> Counter: monotonically increasing (only goes up, resets on restart). Use for: requests total, errors total, bytes sent. Gauge: can go up or down. Use for: temperature, CPU usage, queue depth, memory. You use rate() on counters to get per-second change.

**Q4: Write a PromQL query for the 99th percentile latency.**
> `histogram_quantile(0.99, sum by (le) (rate(http_request_duration_seconds_bucket[5m])))` — This calculates the 99th percentile from histogram buckets, using a 5-minute rate window. The `le` label (less than or equal) is required for histogram_quantile.

**Q5: How do you handle alert fatigue?**
> Set appropriate thresholds with `for` duration (avoid transient spikes). Use severity levels correctly (page for critical only). Implement inhibition rules (suppress child alerts when parent fires). Group related alerts. Include runbook links. Review alerts quarterly — delete ones no one acts on. Route alerts to the right team.

**Q6: Explain SLI, SLO, and SLA with an example.**
> SLI: measured value (99.7% of requests under 200ms). SLO: internal target (99.9% of requests must be under 200ms). SLA: contractual promise with consequences (99.5% uptime guaranteed or customer gets credits). Error budget = 100% - SLO = allowed failure budget.

**Q7: What is the difference between ELK and Loki for logging?**
> ELK (Elasticsearch + Logstash + Kibana): full-text indexing, powerful search, complex queries, but expensive at scale (indexes everything). Loki: indexes only labels (like Prometheus), stores log content compressed, much cheaper, but less powerful full-text search. Choose Loki for Kubernetes-native, cost-effective logging with Grafana.

**Q8: How would you monitor a Kubernetes cluster?**
> Install kube-prometheus-stack (Helm chart) which includes: Prometheus (metrics), Grafana (dashboards), Alertmanager (alerts), Node Exporter (host metrics), kube-state-metrics (K8s objects). Add Loki for logs, Tempo for traces. Monitor: pod restarts, resource usage, node conditions, deployment replicas, PVC usage, network policies.

**Q9: What is distributed tracing and why is it important?**
> Distributed tracing tracks a request's journey across multiple microservices. Each service creates a "span" with timing data. Together, spans form a "trace" showing the full request path. It's essential for: identifying bottlenecks, understanding service dependencies, debugging latency issues in distributed systems.

**Q10: Explain the Golden Signals and when to use them.**
> From Google's SRE book: Latency (request duration), Traffic (requests per second), Errors (failure rate), Saturation (resource fullness). Use for any service/component. They answer: is the service healthy (errors), handling load (traffic), performing well (latency), and approaching limits (saturation)?

---

## Section 8: Free Resources with Links

### AWS Resources

1. **AWS Free Tier** — https://aws.amazon.com/free/ — 12 months of free services
2. **AWS Skill Builder** — https://explore.skillbuilder.aws/ — Free learning paths and labs
3. **AWS Well-Architected Labs** — https://wellarchitectedlabs.com/ — Hands-on labs
4. **AWS Documentation** — https://docs.aws.amazon.com/ — Comprehensive reference
5. **AWS Architecture Center** — https://aws.amazon.com/architecture/ — Reference architectures

### Azure Resources

6. **Azure Free Account** — https://azure.microsoft.com/free/ — $200 credit + 12 months free
7. **Microsoft Learn** — https://learn.microsoft.com/training/ — Free learning paths with sandboxes
8. **Azure Architecture Center** — https://learn.microsoft.com/azure/architecture/ — Reference architectures
9. **AZ-104 Learning Path** — https://learn.microsoft.com/certifications/azure-administrator/ — Free study material
10. **Azure Friday** — https://learn.microsoft.com/shows/azure-friday/ — Video series

### Monitoring Resources

11. **Prometheus Documentation** — https://prometheus.io/docs/ — Official docs and PromQL reference
12. **Grafana Tutorials** — https://grafana.com/tutorials/ — Hands-on guides
13. **Google SRE Book (free)** — https://sre.google/sre-book/table-of-contents/ — Monitoring chapters essential
14. **Awesome Prometheus Alerts** — https://samber.github.io/awesome-prometheus-alerts/ — Ready-to-use alert rules
15. **PromQL Cheat Sheet** — https://promlabs.com/promql-cheat-sheet/ — Quick reference
16. **OpenTelemetry Docs** — https://opentelemetry.io/docs/ — Instrumentation standard
17. **Grafana Play** — https://play.grafana.org/ — Explore Grafana without installation
18. **KillerCoda (free labs)** — https://killercoda.com/ — Hands-on Kubernetes and monitoring scenarios

---

## Section 9: Certification Prep

### AWS Solutions Architect Associate (SAA-C03)

**Exam Overview:**
- Duration: 130 minutes, 65 questions
- Passing: 720/1000
- Cost: $150 USD
- Domains: Design Resilient (26%), High-Performance (24%), Secure (30%), Cost-Optimized (20%)

**Study Strategy:**
1. **Week 1-2:** Core services (EC2, S3, VPC, IAM, RDS)
2. **Week 3-4:** Advanced services (ECS, Lambda, CloudFront, Route 53)
3. **Week 5-6:** Architecture patterns (multi-tier, serverless, DR strategies)
4. **Week 7-8:** Practice exams and weak area review

**Key Topics to Master:**
- VPC design (subnets, routing, security groups, NACLs, endpoints)
- S3 storage classes and lifecycle policies
- IAM policies, roles, cross-account access
- High availability patterns (Multi-AZ, multi-region)
- Serverless architectures (Lambda, API Gateway, DynamoDB)
- Cost optimization (Reserved Instances, Spot, rightsizing)
- Migration strategies (6 R's: Rehost, Replatform, Refactor, Repurchase, Retain, Retire)

**Top Tips:**
- Every question has a "most cost-effective" or "most available" answer — read carefully
- "Least operational overhead" usually means managed/serverless services
- Security + availability are NEVER wrong to prioritize
- Practice reading architecture diagrams
- Take at least 3 full practice exams (Tutorials Dojo recommended)

---

### Azure Administrator (AZ-104)

**Exam Overview:**
- Duration: 120 minutes, 40-60 questions
- Passing: 700/1000
- Cost: $165 USD
- Domains: Identity (20%), Governance (15%), Storage (15%), Compute (20%), Networking (15%), Monitor (15%)

**Study Strategy:**
1. **Week 1-2:** Identity & Governance (Entra ID, RBAC, Policy, Management Groups)
2. **Week 3-4:** Compute & Storage (VMs, VMSS, AKS, Blob, Disks, Files)
3. **Week 5-6:** Networking (VNet, NSG, Peering, Load Balancer, App Gateway, VPN)
4. **Week 7-8:** Monitoring & Practice (Azure Monitor, Log Analytics, Backup, practice exams)

**Key Topics to Master:**
- Entra ID: users, groups, roles, MFA, conditional access
- RBAC: built-in roles, custom roles, role assignments at different scopes
- Azure Policy: built-in policies, initiatives, compliance
- VNet: subnets, NSG, peering, private endpoints, DNS
- Storage: access tiers, lifecycle, SAS tokens, encryption
- VM: availability sets vs zones, VMSS, extensions
- AKS: node pools, networking, scaling, upgrade
- Azure Monitor: metrics, logs, KQL queries, alerts, action groups

**Top Tips:**
- Microsoft Learn has FREE labs with Azure sandbox — use them
- AZ-104 is hands-on heavy — you need to practice in a real Azure subscription
- Know the az CLI syntax (many questions involve CLI commands)
- Understand Azure pricing tiers (Basic vs Standard vs Premium for each service)
- Network security questions are common — know NSG priority rules
- KQL queries appear in the exam — practice with Log Analytics
- Questions often have "case study" format — read the entire scenario carefully

---

### GCC-Specific Certification Advice

- **Azure certifications are highly valued** in GCC markets (Saudi, UAE, Qatar)
- **Start with AZ-104** if targeting GCC enterprise/government roles
- **AWS SAA** is valuable for startups and tech companies in the region
- Many GCC companies require **at least one cloud certification** for senior DevOps roles
- Consider **AZ-400 (DevOps Engineer Expert)** as a follow-up to AZ-104
- **Security certifications** (AZ-500, AWS Security Specialty) are premium in GCC due to compliance focus

---

## Summary & Learning Path

```
Week 1-2: Cloud Fundamentals + AWS Core (IAM, VPC, EC2, S3)
Week 3-4: AWS Advanced + Azure Core (Azure AD, VNet, VMs, AKS)
Week 5-6: Azure Advanced + Monitoring Basics (Prometheus, Grafana)
Week 7-8: Advanced Monitoring (Alerting, Logging, Tracing, Incident Management)

Daily Practice:
- 30 min: Read documentation
- 1 hour: Hands-on lab (free tier / sandbox)
- 30 min: PromQL / KQL queries
- 15 min: Review interview questions
```

---

*End of Month 5: Cloud Computing & Monitoring*
*Next: Month 6 — Infrastructure as Code (Terraform, Ansible, CloudFormation, Bicep)*
