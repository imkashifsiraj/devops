# Month 3 - Part 1: Terraform (Infrastructure as Code)

> **DevOps Upskilling Program — Job Market Focus**
> Comprehensive study material covering Terraform fundamentals through advanced patterns.

---

## 1. What is Infrastructure as Code (IaC)? Why IaC?

### Definition
Infrastructure as Code (IaC) is the practice of managing and provisioning computing infrastructure through machine-readable configuration files rather than manual processes or interactive tools.

### Manual vs IaC Approach

| Aspect | Manual | IaC |
|--------|--------|-----|
| Speed | Slow, click-by-click | Fast, automated |
| Consistency | Prone to human error | Repeatable, identical every time |
| Documentation | Separate docs needed | Code IS the documentation |
| Version Control | Not possible | Full git history |
| Scalability | Doesn't scale | Scales infinitely |
| Audit Trail | Limited | Complete via git commits |
| Disaster Recovery | Rebuild from memory | Re-run the code |
| Cost | High (engineer time) | Low after initial setup |

### Declarative vs Imperative

**Declarative (What):** You describe the desired end state. The tool figures out how to get there.
```hcl
# Terraform (Declarative) - "I want 3 EC2 instances"
resource "aws_instance" "web" {
  count         = 3
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t3.micro"
}
```

**Imperative (How):** You describe the exact steps to achieve the result.
```python
# Pulumi/SDK approach (Imperative) - "Create instances one by one"
for i in range(3):
    ec2.create_instance(ami="ami-0c55b159cbfafe1f0", type="t3.micro")
```

### IaC Tool Comparison

| Feature | Terraform | CloudFormation | Pulumi | Bicep |
|---------|-----------|----------------|--------|-------|
| Provider | Multi-cloud | AWS only | Multi-cloud | Azure only |
| Language | HCL | JSON/YAML | Python/TS/Go | Bicep DSL |
| State | Self-managed | AWS-managed | Self-managed | Azure-managed |
| Learning Curve | Medium | Medium | Low (if you know the language) | Low |
| Community | Massive | Large (AWS) | Growing | Growing |
| Open Source | Yes (BSL) | No | Yes | Yes |
| GCC Market Demand | Very High | High | Medium | Medium |

**Why Terraform wins for GCC market:**
- Multi-cloud is common (AWS + Azure deployments)
- Most DevOps job postings list Terraform
- Vendor-neutral certification (HashiCorp Terraform Associate)
- Massive module ecosystem

---

## 2. Terraform Architecture

### How Terraform Works: The Core Workflow

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│  terraform  │───▶│  terraform  │───▶│  terraform  │
│    init     │    │    plan     │    │    apply    │
└─────────────┘    └─────────────┘    └─────────────┘
       │                  │                  │
       ▼                  ▼                  ▼
 Download providers  Compare desired   Execute changes
 Initialize backend  vs current state  Update state file
```

**terraform init:**
- Downloads provider plugins
- Initializes the backend (state storage)
- Downloads modules
- Run once per new config or backend change

**terraform plan:**
- Reads current state
- Compares with desired configuration
- Shows what will be created/modified/destroyed
- Does NOT make any changes

**terraform apply:**
- Executes the plan
- Creates/modifies/destroys resources
- Updates the state file
- Shows output values

### Providers, Resources, and Data Sources

```hcl
# Provider: Plugin that talks to an API
provider "aws" {
  region = "me-south-1"  # Bahrain - GCC region
}

# Resource: Infrastructure object to create/manage
resource "aws_instance" "app_server" {
  ami           = "ami-0123456789abcdef0"
  instance_type = "t3.medium"

  tags = {
    Name        = "app-server"
    Environment = "production"
  }
}

# Data Source: Query existing infrastructure (read-only)
data "aws_ami" "ubuntu" {
  most_recent = true
  owners      = ["099720109477"]  # Canonical

  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-jammy-22.04-amd64-server-*"]
  }
}

# Using the data source in a resource
resource "aws_instance" "web" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = "t3.micro"
}
```

### State File and Its Importance

The state file (`terraform.tfstate`) is a JSON file that maps your configuration to real-world resources.

**Why state matters:**
1. Maps config to real resource IDs
2. Tracks metadata and dependencies
3. Enables plan/diff calculations
4. Improves performance (caches attributes)
5. Enables team collaboration (remote state)

### Terraform Registry

The [Terraform Registry](https://registry.terraform.io/) provides:
- **Providers:** AWS, Azure, GCP, Kubernetes, etc.
- **Modules:** Pre-built, reusable configurations
- **Policy Libraries:** Sentinel policies

---

## 3. HCL (HashiCorp Configuration Language) Deep Dive

### Variables

```hcl
# variables.tf

# String variable
variable "environment" {
  description = "Deployment environment"
  type        = string
  default     = "development"
}

# Number variable
variable "instance_count" {
  description = "Number of instances to create"
  type        = number
  default     = 2
}

# Boolean variable
variable "enable_monitoring" {
  description = "Enable CloudWatch monitoring"
  type        = bool
  default     = true
}

# List variable
variable "availability_zones" {
  description = "AZs to deploy into"
  type        = list(string)
  default     = ["me-south-1a", "me-south-1b", "me-south-1c"]
}

# Map variable
variable "instance_types" {
  description = "Instance types per environment"
  type        = map(string)
  default = {
    development = "t3.micro"
    staging     = "t3.small"
    production  = "t3.medium"
  }
}

# Object variable
variable "vpc_config" {
  description = "VPC configuration"
  type = object({
    cidr_block           = string
    enable_dns_hostnames = bool
    enable_dns_support   = bool
    tags                 = map(string)
  })
  default = {
    cidr_block           = "10.0.0.0/16"
    enable_dns_hostnames = true
    enable_dns_support   = true
    tags = {
      Project = "my-app"
    }
  }
}

# Tuple variable (fixed length, mixed types)
variable "rule" {
  type    = tuple([string, number, string])
  default = ["tcp", 443, "0.0.0.0/0"]
}

# Set variable (unique values, unordered)
variable "allowed_ports" {
  type    = set(number)
  default = [80, 443, 8080]
}
```

### Variable Validation

```hcl
variable "environment" {
  type        = string
  description = "Must be dev, staging, or prod"

  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "Environment must be one of: dev, staging, prod."
  }
}

variable "cidr_block" {
  type        = string
  description = "VPC CIDR block"

  validation {
    condition     = can(cidrhost(var.cidr_block, 0))
    error_message = "Must be a valid CIDR block (e.g., 10.0.0.0/16)."
  }
}

variable "instance_type" {
  type = string

  validation {
    condition     = regex("^t3\\.", var.instance_type) != ""
    error_message = "Only t3 instance types are allowed."
  }
}
```

### Type Constraints

```hcl
# any - accepts any type
variable "flexible" {
  type = any
}

# Complex nested types
variable "applications" {
  type = list(object({
    name     = string
    port     = number
    protocol = string
    health_check = object({
      path     = string
      interval = number
    })
  }))
}

# Optional attributes (Terraform 1.3+)
variable "settings" {
  type = object({
    name     = string
    enabled  = optional(bool, true)       # default: true
    timeout  = optional(number, 30)       # default: 30
    tags     = optional(map(string), {})  # default: empty map
  })
}
```

### Outputs

```hcl
# outputs.tf

output "instance_id" {
  description = "ID of the EC2 instance"
  value       = aws_instance.app_server.id
}

output "instance_public_ip" {
  description = "Public IP address"
  value       = aws_instance.app_server.public_ip
}

# Sensitive output (hidden in CLI)
output "database_password" {
  description = "RDS master password"
  value       = aws_db_instance.main.password
  sensitive   = true
}

# Conditional output
output "load_balancer_dns" {
  description = "ALB DNS name (only in production)"
  value       = var.environment == "prod" ? aws_lb.main[0].dns_name : "N/A"
}
```

### Locals

```hcl
locals {
  # Computed values
  name_prefix = "${var.project}-${var.environment}"

  # Common tags applied everywhere
  common_tags = {
    Project     = var.project
    Environment = var.environment
    ManagedBy   = "terraform"
    Team        = "platform"
    CostCenter  = "engineering"
  }

  # Complex computation
  subnet_cidrs = [for i in range(var.subnet_count) : cidrsubnet(var.vpc_cidr, 8, i)]

  # Conditional logic
  instance_type = var.environment == "prod" ? "t3.large" : "t3.micro"
}

# Usage
resource "aws_instance" "app" {
  instance_type = local.instance_type
  tags          = merge(local.common_tags, { Name = "${local.name_prefix}-app" })
}
```

### Expressions and Functions

```hcl
locals {
  # lookup - Get value from map with default
  instance_type = lookup(var.instance_types, var.environment, "t3.micro")

  # merge - Combine maps (later values override)
  all_tags = merge(local.common_tags, var.extra_tags, { LastModified = timestamp() })

  # flatten - Flatten nested lists
  all_subnets = flatten([var.public_subnets, var.private_subnets])

  # format - String formatting
  bucket_name = format("%s-%s-%s", var.project, var.environment, var.region)

  # join - Join list elements with separator
  subnet_ids_str = join(",", aws_subnet.private[*].id)

  # split - Split string into list
  cidr_parts = split(".", var.vpc_cidr)

  # try - Return first expression that doesn't error
  db_port = try(var.custom_settings.database.port, 5432)

  # can - Test if expression is valid (returns bool)
  has_custom_port = can(var.custom_settings.database.port)

  # coalesce - First non-null/non-empty value
  region = coalesce(var.region, data.aws_region.current.name)

  # zipmap - Create map from two lists
  instance_map = zipmap(var.instance_names, aws_instance.servers[*].id)

  # range - Generate number sequence
  port_list = range(8080, 8085)  # [8080, 8081, 8082, 8083, 8084]

  # templatefile - Render template
  user_data = templatefile("${path.module}/userdata.tpl", {
    environment = var.environment
    app_port    = var.app_port
  })
}
```

### Dynamic Blocks

```hcl
# Without dynamic block (repetitive)
resource "aws_security_group" "example" {
  name = "example"

  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  ingress {
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

# With dynamic block (DRY)
variable "ingress_rules" {
  default = [
    { port = 80,  cidr = "0.0.0.0/0", description = "HTTP" },
    { port = 443, cidr = "0.0.0.0/0", description = "HTTPS" },
    { port = 8080, cidr = "10.0.0.0/8", description = "App" },
  ]
}

resource "aws_security_group" "example" {
  name        = "dynamic-sg"
  description = "Security group with dynamic rules"
  vpc_id      = aws_vpc.main.id

  dynamic "ingress" {
    for_each = var.ingress_rules
    content {
      from_port   = ingress.value.port
      to_port     = ingress.value.port
      protocol    = "tcp"
      cidr_blocks = [ingress.value.cidr]
      description = ingress.value.description
    }
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}
```

### Conditional Expressions

```hcl
# Simple ternary
resource "aws_instance" "app" {
  instance_type = var.environment == "prod" ? "t3.large" : "t3.micro"

  # Conditional resource creation
  count = var.create_instance ? 1 : 0
}

# Conditional with null (omit attribute)
resource "aws_instance" "app" {
  ami           = var.ami_id
  instance_type = var.instance_type
  key_name      = var.environment == "prod" ? var.key_name : null
}
```

### for_each vs count (Detailed Comparison)

```hcl
# ===== COUNT =====
# Creates indexed resources: aws_instance.server[0], aws_instance.server[1]
resource "aws_instance" "server" {
  count         = 3
  ami           = "ami-0123456789"
  instance_type = "t3.micro"
  tags = {
    Name = "server-${count.index}"
  }
}

# Problem with count: removing item from middle reindexes everything
# If you delete server[1], server[2] becomes server[1] = DESTROY + RECREATE

# ===== FOR_EACH with set =====
resource "aws_instance" "server" {
  for_each      = toset(["web", "app", "db"])
  ami           = "ami-0123456789"
  instance_type = "t3.micro"
  tags = {
    Name = "server-${each.key}"
  }
}
# Creates: aws_instance.server["web"], aws_instance.server["app"], aws_instance.server["db"]
# Removing "app" only destroys that one instance!

# ===== FOR_EACH with map =====
variable "instances" {
  default = {
    web = { type = "t3.small", az = "me-south-1a" }
    app = { type = "t3.medium", az = "me-south-1b" }
    db  = { type = "r5.large", az = "me-south-1a" }
  }
}

resource "aws_instance" "server" {
  for_each      = var.instances
  ami           = "ami-0123456789"
  instance_type = each.value.type
  availability_zone = each.value.az
  tags = {
    Name = "${each.key}-server"
  }
}
```

**When to use which:**
| Scenario | Use |
|----------|-----|
| Simple numeric scaling | `count` |
| Resources with unique identities | `for_each` |
| Items that may be added/removed | `for_each` |
| Conditional creation (0 or 1) | `count` |
| Map/set of configurations | `for_each` |

### Splat Expressions

```hcl
# Splat expression: shorthand for getting attributes from all instances
output "all_instance_ids" {
  value = aws_instance.server[*].id
}

# Equivalent for_each (use values() since for_each creates a map)
output "all_instance_ids" {
  value = [for instance in aws_instance.server : instance.id]
}

# Nested splat
output "all_private_ips" {
  value = aws_instance.server[*].private_ip
}
```

---

## 4. Provider Configuration

### AWS Provider Setup

```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
  required_version = ">= 1.5.0"
}

# Basic provider
provider "aws" {
  region = "me-south-1"  # Bahrain

  default_tags {
    tags = {
      ManagedBy   = "terraform"
      Environment = var.environment
    }
  }
}
```

### Azure Provider Setup

```hcl
terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 3.0"
    }
  }
}

provider "azurerm" {
  features {
    resource_group {
      prevent_deletion_if_contains_resources = true
    }
    key_vault {
      purge_soft_delete_on_destroy = false
    }
  }

  subscription_id = var.subscription_id
  tenant_id       = var.tenant_id
}
```

### Multiple Provider Configurations (Aliases)

```hcl
# Default provider - Bahrain
provider "aws" {
  region = "me-south-1"
}

# Alias for UAE
provider "aws" {
  alias  = "uae"
  region = "me-central-1"
}

# Alias for us-east-1 (for global resources like CloudFront/ACM)
provider "aws" {
  alias  = "virginia"
  region = "us-east-1"
}

# Using aliased provider
resource "aws_instance" "uae_server" {
  provider      = aws.uae
  ami           = "ami-uae-specific"
  instance_type = "t3.micro"
}

resource "aws_acm_certificate" "cert" {
  provider          = aws.virginia
  domain_name       = "example.com"
  validation_method = "DNS"
}
```

### Version Constraints

```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"      # >= 5.0, < 6.0 (recommended)
    }
    azurerm = {
      source  = "hashicorp/azurerm"
      version = ">= 3.50, < 4.0"  # Range constraint
    }
    random = {
      source  = "hashicorp/random"
      version = "= 3.5.1"    # Exact version (strict)
    }
  }
}
# Operators: = (exact), != (not), >, >=, <, <=, ~> (pessimistic)
# ~> 5.0 means >= 5.0, < 6.0
# ~> 5.1 means >= 5.1, < 5.2
```

---

## 5. State Management (CRITICAL)

### What is State and Why It Matters

State is Terraform's record of what infrastructure it manages. Without state, Terraform cannot:
- Know what exists in the real world
- Calculate what needs to change
- Track resource dependencies
- Map configuration to actual resources

### Local vs Remote State

```hcl
# Local state (default) - terraform.tfstate in current directory
# NEVER use in production or team environments

# Remote state - stored in a shared backend
terraform {
  backend "s3" {
    bucket = "my-terraform-state"
    key    = "production/terraform.tfstate"
    region = "me-south-1"
  }
}
```

### S3 + DynamoDB Backend (Complete Setup)

**Step 1: Create the backend infrastructure (bootstrap)**
```hcl
# backend-setup/main.tf - Run this FIRST with local state
provider "aws" {
  region = "me-south-1"
}

resource "aws_s3_bucket" "terraform_state" {
  bucket = "mycompany-terraform-state-prod"

  lifecycle {
    prevent_destroy = true
  }
}

resource "aws_s3_bucket_versioning" "terraform_state" {
  bucket = aws_s3_bucket.terraform_state.id
  versioning_configuration {
    status = "Enabled"
  }
}

resource "aws_s3_bucket_server_side_encryption_configuration" "terraform_state" {
  bucket = aws_s3_bucket.terraform_state.id
  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm = "aws:kms"
    }
  }
}

resource "aws_s3_bucket_public_access_block" "terraform_state" {
  bucket                  = aws_s3_bucket.terraform_state.id
  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}

resource "aws_dynamodb_table" "terraform_locks" {
  name         = "terraform-state-locks"
  billing_mode = "PAY_PER_REQUEST"
  hash_key     = "LockID"

  attribute {
    name = "LockID"
    type = "S"
  }

  tags = {
    Name = "Terraform State Lock Table"
  }
}
```

**Step 2: Configure the backend in your project**
```hcl
terraform {
  backend "s3" {
    bucket         = "mycompany-terraform-state-prod"
    key            = "environments/production/terraform.tfstate"
    region         = "me-south-1"
    dynamodb_table = "terraform-state-locks"
    encrypt        = true
    kms_key_id     = "alias/terraform-state"
  }
}
```

### Azure Blob Backend

```hcl
terraform {
  backend "azurerm" {
    resource_group_name  = "terraform-state-rg"
    storage_account_name = "tfstatecompany"
    container_name       = "tfstate"
    key                  = "production.terraform.tfstate"
  }
}
```

### State Locking

State locking prevents concurrent operations that could corrupt state:
- S3 backend uses DynamoDB for locking
- Azure backend uses blob leases
- If a lock is stuck: `terraform force-unlock LOCK_ID`

### State Commands

```bash
# List all resources in state
terraform state list

# Show details of a specific resource
terraform state show aws_instance.web

# Move/rename a resource in state (refactoring)
terraform state mv aws_instance.web aws_instance.app_server

# Remove a resource from state (stop managing it, don't destroy)
terraform state rm aws_instance.legacy_server

# Pull remote state to local file
terraform state pull > state_backup.json

# Push local state to remote (DANGEROUS)
terraform state push terraform.tfstate

# Replace provider in state
terraform state replace-provider hashicorp/aws registry.example.com/aws
```

### terraform import

```bash
# Import existing resource into Terraform state
terraform import aws_instance.web i-1234567890abcdef0

# Import with for_each key
terraform import 'aws_instance.servers["web"]' i-1234567890abcdef0

# Import module resource
terraform import module.vpc.aws_vpc.this vpc-0123456789
```

```hcl
# Terraform 1.5+ import blocks (declarative)
import {
  to = aws_instance.web
  id = "i-1234567890abcdef0"
}

# Generate config for imported resource
# terraform plan -generate-config-out=generated.tf
```

### State File Security

**Sensitive data in state:** Terraform state contains ALL attribute values including:
- Database passwords
- Private keys
- API tokens
- Any sensitive output

**Protections:**
1. Always encrypt state at rest (S3 encryption, Azure encryption)
2. Restrict access to state bucket (IAM policies)
3. Enable versioning for recovery
4. Never commit state files to git
5. Use `sensitive = true` on variables/outputs

```gitignore
# .gitignore
*.tfstate
*.tfstate.*
.terraform/
*.tfvars  # May contain secrets
```

### State Drift Detection

```bash
# Detect drift: compare state vs real infrastructure
terraform plan -refresh-only

# Apply refresh to update state without changing infra
terraform apply -refresh-only
```

---

## 6. Modules

### What Are Modules and Why Use Them

A module is a container for multiple resources used together. Benefits:
- **Reusability:** Write once, use in multiple environments
- **Encapsulation:** Hide complexity behind a simple interface
- **Consistency:** Enforce standards across teams
- **Maintainability:** Update in one place

### Module Structure

```
modules/
└── vpc/
    ├── main.tf          # Resource definitions
    ├── variables.tf     # Input variables
    ├── outputs.tf       # Output values
    ├── versions.tf      # Provider requirements
    └── README.md        # Documentation
```

### Using Community Modules

```hcl
# AWS VPC module from registry
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "5.1.0"

  name = "my-vpc"
  cidr = "10.0.0.0/16"

  azs             = ["me-south-1a", "me-south-1b", "me-south-1c"]
  private_subnets = ["10.0.1.0/24", "10.0.2.0/24", "10.0.3.0/24"]
  public_subnets  = ["10.0.101.0/24", "10.0.102.0/24", "10.0.103.0/24"]

  enable_nat_gateway = true
  single_nat_gateway = true

  tags = local.common_tags
}

# AWS EKS module
module "eks" {
  source  = "terraform-aws-modules/eks/aws"
  version = "19.15.0"

  cluster_name    = "my-cluster"
  cluster_version = "1.28"
  vpc_id          = module.vpc.vpc_id
  subnet_ids      = module.vpc.private_subnets

  eks_managed_node_groups = {
    general = {
      desired_size = 2
      min_size     = 1
      max_size     = 5
      instance_types = ["t3.medium"]
    }
  }
}
```

### Writing Custom Modules (Complete Example)

```hcl
# modules/web-app/variables.tf
variable "app_name" {
  description = "Application name"
  type        = string
}

variable "environment" {
  description = "Environment (dev/staging/prod)"
  type        = string
}

variable "instance_type" {
  description = "EC2 instance type"
  type        = string
  default     = "t3.micro"
}

variable "vpc_id" {
  description = "VPC ID to deploy into"
  type        = string
}

variable "subnet_ids" {
  description = "Subnet IDs for deployment"
  type        = list(string)
}
```

```hcl
# modules/web-app/main.tf
resource "aws_security_group" "app" {
  name        = "${var.app_name}-${var.environment}-sg"
  description = "Security group for ${var.app_name}"
  vpc_id      = var.vpc_id

  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "${var.app_name}-${var.environment}-sg"
  }
}

resource "aws_instance" "app" {
  count                  = var.environment == "prod" ? 2 : 1
  ami                    = data.aws_ami.amazon_linux.id
  instance_type          = var.instance_type
  subnet_id              = var.subnet_ids[count.index % length(var.subnet_ids)]
  vpc_security_group_ids = [aws_security_group.app.id]

  tags = {
    Name        = "${var.app_name}-${var.environment}-${count.index}"
    Environment = var.environment
  }
}

data "aws_ami" "amazon_linux" {
  most_recent = true
  owners      = ["amazon"]
  filter {
    name   = "name"
    values = ["amzn2-ami-hvm-*-x86_64-gp2"]
  }
}
```

```hcl
# modules/web-app/outputs.tf
output "instance_ids" {
  description = "IDs of created instances"
  value       = aws_instance.app[*].id
}

output "security_group_id" {
  description = "Security group ID"
  value       = aws_security_group.app.id
}

output "private_ips" {
  description = "Private IPs of instances"
  value       = aws_instance.app[*].private_ip
}
```

```hcl
# Root module usage
module "web_app" {
  source = "./modules/web-app"

  app_name      = "payment-api"
  environment   = "prod"
  instance_type = "t3.medium"
  vpc_id        = module.vpc.vpc_id
  subnet_ids    = module.vpc.private_subnets
}
```

### Module Sources

```hcl
# Local path
module "vpc" {
  source = "./modules/vpc"
}

# Terraform Registry
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "5.1.0"
}

# GitHub
module "vpc" {
  source = "github.com/company/terraform-modules//vpc?ref=v1.2.0"
}

# S3 bucket
module "vpc" {
  source = "s3::https://s3-me-south-1.amazonaws.com/my-modules/vpc.zip"
}

# Git over SSH
module "vpc" {
  source = "git::ssh://git@github.com/company/modules.git//vpc?ref=v1.0.0"
}
```

### Module Versioning

```hcl
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "~> 5.0"   # >= 5.0, < 6.0

  # Pin to exact for production stability
  # version = "5.1.2"
}
```

---

## 7. Resource Lifecycle

```hcl
resource "aws_instance" "web" {
  ami           = var.ami_id
  instance_type = var.instance_type

  lifecycle {
    # Create new resource before destroying old (zero downtime)
    create_before_destroy = true

    # Prevent accidental deletion
    prevent_destroy = true

    # Ignore changes made outside Terraform
    ignore_changes = [
      tags["LastModified"],
      ami,  # Ignore AMI changes (managed by another process)
    ]

    # Replace when another resource changes
    replace_triggered_by = [
      aws_security_group.web.id
    ]
  }
}

# Precondition and postcondition (Terraform 1.2+)
resource "aws_instance" "web" {
  ami           = var.ami_id
  instance_type = var.instance_type

  lifecycle {
    precondition {
      condition     = data.aws_ami.selected.architecture == "x86_64"
      error_message = "AMI must be x86_64 architecture."
    }

    postcondition {
      condition     = self.public_ip != ""
      error_message = "Instance must have a public IP."
    }
  }
}
```

---

## 8. Workspaces

### What Are Workspaces

Workspaces allow multiple state files for the same configuration. Each workspace has its own state.

```bash
# List workspaces
terraform workspace list

# Create new workspace
terraform workspace new staging

# Switch workspace
terraform workspace select production

# Show current workspace
terraform workspace show

# Delete workspace
terraform workspace delete staging
```

### Using Workspaces in Configuration

```hcl
locals {
  environment = terraform.workspace

  instance_type = {
    default = "t3.micro"
    staging = "t3.small"
    prod    = "t3.large"
  }
}

resource "aws_instance" "app" {
  instance_type = lookup(local.instance_type, terraform.workspace, "t3.micro")

  tags = {
    Environment = terraform.workspace
  }
}
```

### When to Use Workspaces vs Separate State Files

| Use Workspaces | Use Separate Directories |
|---|---|
| Same config, different params | Different infrastructure per env |
| Dev/staging with same structure | Prod needs extra resources |
| Quick experimentation | Team isolation needed |
| Simple environments | Complex access control |

**Recommendation:** For GCC enterprise projects, prefer separate directories with shared modules over workspaces for better isolation and access control.

---

## 9. Terraform with CI/CD

### Pipeline Structure

```
PR Created → terraform plan → Comment results on PR
PR Merged  → terraform apply → Deploy infrastructure
```

### GitLab CI Example

```yaml
# .gitlab-ci.yml
stages:
  - validate
  - plan
  - apply

variables:
  TF_ROOT: ${CI_PROJECT_DIR}/infrastructure
  TF_STATE_NAME: production

.terraform-base:
  image: hashicorp/terraform:1.6
  before_script:
    - cd ${TF_ROOT}
    - terraform init -backend-config="address=${CI_API_V4_URL}/projects/${CI_PROJECT_ID}/terraform/state/${TF_STATE_NAME}"

validate:
  extends: .terraform-base
  stage: validate
  script:
    - terraform validate
    - terraform fmt -check
  rules:
    - if: $CI_MERGE_REQUEST_IID

plan:
  extends: .terraform-base
  stage: plan
  script:
    - terraform plan -out=plan.tfplan
    - terraform show -json plan.tfplan > plan.json
  artifacts:
    paths:
      - ${TF_ROOT}/plan.tfplan
    expire_in: 1 week
  rules:
    - if: $CI_MERGE_REQUEST_IID

apply:
  extends: .terraform-base
  stage: apply
  script:
    - terraform apply -auto-approve plan.tfplan
  dependencies:
    - plan
  rules:
    - if: $CI_COMMIT_BRANCH == "main"
  when: manual
  environment:
    name: production
```

### Azure DevOps Example

```yaml
# azure-pipelines.yml
trigger:
  branches:
    include:
      - main

pool:
  vmImage: 'ubuntu-latest'

variables:
  - group: terraform-secrets
  - name: tfWorkingDir
    value: '$(System.DefaultWorkingDirectory)/infrastructure'

stages:
  - stage: Plan
    jobs:
      - job: TerraformPlan
        steps:
          - task: TerraformInstaller@0
            inputs:
              terraformVersion: '1.6.0'

          - task: TerraformTaskV4@4
            displayName: 'Terraform Init'
            inputs:
              provider: 'azurerm'
              command: 'init'
              workingDirectory: '$(tfWorkingDir)'
              backendServiceArm: 'Azure-Service-Connection'
              backendAzureRmResourceGroupName: 'terraform-state-rg'
              backendAzureRmStorageAccountName: 'tfstatecompany'
              backendAzureRmContainerName: 'tfstate'
              backendAzureRmKey: 'production.tfstate'

          - task: TerraformTaskV4@4
            displayName: 'Terraform Plan'
            inputs:
              provider: 'azurerm'
              command: 'plan'
              workingDirectory: '$(tfWorkingDir)'
              commandOptions: '-out=tfplan'
              environmentServiceNameAzureRM: 'Azure-Service-Connection'

  - stage: Apply
    dependsOn: Plan
    condition: and(succeeded(), eq(variables['Build.SourceBranch'], 'refs/heads/main'))
    jobs:
      - deployment: TerraformApply
        environment: 'production'
        strategy:
          runOnce:
            deploy:
              steps:
                - task: TerraformTaskV4@4
                  displayName: 'Terraform Apply'
                  inputs:
                    provider: 'azurerm'
                    command: 'apply'
                    workingDirectory: '$(tfWorkingDir)'
                    commandOptions: 'tfplan'
                    environmentServiceNameAzureRM: 'Azure-Service-Connection'
```

### Security Considerations

1. **Never store secrets in Terraform files** — use environment variables or vault
2. **Use least-privilege IAM** for CI/CD service accounts
3. **Require plan approval** before apply in production
4. **Encrypt state** at rest and in transit
5. **Use OIDC** for cloud authentication (no long-lived credentials)
6. **Scan with tfsec/checkov** in pipeline

---

## 10. Complete Real-World Examples

### AWS VPC with Public/Private Subnets

```hcl
# vpc/main.tf
resource "aws_vpc" "main" {
  cidr_block           = var.vpc_cidr
  enable_dns_hostnames = true
  enable_dns_support   = true

  tags = {
    Name = "${var.project}-${var.environment}-vpc"
  }
}

resource "aws_internet_gateway" "main" {
  vpc_id = aws_vpc.main.id
  tags   = { Name = "${var.project}-${var.environment}-igw" }
}

resource "aws_subnet" "public" {
  count                   = length(var.public_subnet_cidrs)
  vpc_id                  = aws_vpc.main.id
  cidr_block              = var.public_subnet_cidrs[count.index]
  availability_zone       = var.azs[count.index]
  map_public_ip_on_launch = true

  tags = {
    Name = "${var.project}-public-${var.azs[count.index]}"
    Tier = "public"
  }
}

resource "aws_subnet" "private" {
  count             = length(var.private_subnet_cidrs)
  vpc_id            = aws_vpc.main.id
  cidr_block        = var.private_subnet_cidrs[count.index]
  availability_zone = var.azs[count.index]

  tags = {
    Name = "${var.project}-private-${var.azs[count.index]}"
    Tier = "private"
  }
}

resource "aws_eip" "nat" {
  domain = "vpc"
  tags   = { Name = "${var.project}-nat-eip" }
}

resource "aws_nat_gateway" "main" {
  allocation_id = aws_eip.nat.id
  subnet_id     = aws_subnet.public[0].id
  tags          = { Name = "${var.project}-nat-gw" }
}

resource "aws_route_table" "public" {
  vpc_id = aws_vpc.main.id
  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.main.id
  }
  tags = { Name = "${var.project}-public-rt" }
}

resource "aws_route_table" "private" {
  vpc_id = aws_vpc.main.id
  route {
    cidr_block     = "0.0.0.0/0"
    nat_gateway_id = aws_nat_gateway.main.id
  }
  tags = { Name = "${var.project}-private-rt" }
}

resource "aws_route_table_association" "public" {
  count          = length(var.public_subnet_cidrs)
  subnet_id      = aws_subnet.public[count.index].id
  route_table_id = aws_route_table.public.id
}

resource "aws_route_table_association" "private" {
  count          = length(var.private_subnet_cidrs)
  subnet_id      = aws_subnet.private[count.index].id
  route_table_id = aws_route_table.private.id
}
```

### S3 Bucket with Policies

```hcl
resource "aws_s3_bucket" "app_data" {
  bucket = "${var.project}-${var.environment}-data"

  tags = {
    Name        = "${var.project}-data"
    Environment = var.environment
  }
}

resource "aws_s3_bucket_versioning" "app_data" {
  bucket = aws_s3_bucket.app_data.id
  versioning_configuration {
    status = "Enabled"
  }
}

resource "aws_s3_bucket_server_side_encryption_configuration" "app_data" {
  bucket = aws_s3_bucket.app_data.id
  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm = "aws:kms"
    }
    bucket_key_enabled = true
  }
}

resource "aws_s3_bucket_public_access_block" "app_data" {
  bucket                  = aws_s3_bucket.app_data.id
  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}

resource "aws_s3_bucket_lifecycle_configuration" "app_data" {
  bucket = aws_s3_bucket.app_data.id

  rule {
    id     = "transition-to-ia"
    status = "Enabled"

    transition {
      days          = 30
      storage_class = "STANDARD_IA"
    }
    transition {
      days          = 90
      storage_class = "GLACIER"
    }
    expiration {
      days = 365
    }
  }
}
```

### Security Groups

```hcl
resource "aws_security_group" "web" {
  name        = "${var.project}-web-sg"
  description = "Security group for web servers"
  vpc_id      = aws_vpc.main.id

  tags = { Name = "${var.project}-web-sg" }
}

resource "aws_security_group_rule" "web_ingress_http" {
  type              = "ingress"
  from_port         = 80
  to_port           = 80
  protocol          = "tcp"
  cidr_blocks       = ["0.0.0.0/0"]
  security_group_id = aws_security_group.web.id
  description       = "HTTP from internet"
}

resource "aws_security_group_rule" "web_ingress_https" {
  type              = "ingress"
  from_port         = 443
  to_port           = 443
  protocol          = "tcp"
  cidr_blocks       = ["0.0.0.0/0"]
  security_group_id = aws_security_group.web.id
  description       = "HTTPS from internet"
}

resource "aws_security_group_rule" "web_egress" {
  type              = "egress"
  from_port         = 0
  to_port           = 0
  protocol          = "-1"
  cidr_blocks       = ["0.0.0.0/0"]
  security_group_id = aws_security_group.web.id
  description       = "All outbound"
}

# App SG - only accessible from web SG
resource "aws_security_group" "app" {
  name        = "${var.project}-app-sg"
  description = "Security group for app servers"
  vpc_id      = aws_vpc.main.id

  ingress {
    from_port       = 8080
    to_port         = 8080
    protocol        = "tcp"
    security_groups = [aws_security_group.web.id]
    description     = "App port from web tier"
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = { Name = "${var.project}-app-sg" }
}
```

### Azure Resource Group + VNet + AKS

```hcl
resource "azurerm_resource_group" "main" {
  name     = "${var.project}-${var.environment}-rg"
  location = "UAE North"

  tags = {
    Environment = var.environment
    ManagedBy   = "terraform"
  }
}

resource "azurerm_virtual_network" "main" {
  name                = "${var.project}-vnet"
  address_space       = ["10.0.0.0/16"]
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name
}

resource "azurerm_subnet" "aks" {
  name                 = "aks-subnet"
  resource_group_name  = azurerm_resource_group.main.name
  virtual_network_name = azurerm_virtual_network.main.name
  address_prefixes     = ["10.0.1.0/24"]
}

resource "azurerm_kubernetes_cluster" "main" {
  name                = "${var.project}-aks"
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name
  dns_prefix          = var.project
  kubernetes_version  = "1.28"

  default_node_pool {
    name           = "default"
    node_count     = 3
    vm_size        = "Standard_D2s_v3"
    vnet_subnet_id = azurerm_subnet.aks.id
  }

  identity {
    type = "SystemAssigned"
  }

  network_profile {
    network_plugin = "azure"
    service_cidr   = "10.1.0.0/16"
    dns_service_ip = "10.1.0.10"
  }

  tags = {
    Environment = var.environment
  }
}
```

---

## 11. Best Practices (25 Essential Rules)

### Code Organization
1. **Use consistent file structure:** `main.tf`, `variables.tf`, `outputs.tf`, `versions.tf`, `locals.tf`
2. **Separate environments:** Use directories (`environments/dev/`, `environments/prod/`) not just workspaces
3. **Keep modules small:** One module = one logical component (VPC, EKS, RDS)
4. **Use a `terraform.tfvars` per environment** for variable values

### Naming Conventions
5. **Use snake_case** for all Terraform identifiers (resources, variables, outputs)
6. **Prefix resources** with project/app name: `myapp_web_sg` not just `web_sg`
7. **Use meaningful names:** `aws_instance.payment_api` not `aws_instance.instance1`
8. **Tag everything** with at minimum: Name, Environment, Team, ManagedBy, CostCenter

### Tagging Strategy
9. **Enforce tags with default_tags** in provider block
10. **Use locals for common tags** and merge with resource-specific tags
11. **Include cost allocation tags** for FinOps (GCC orgs love this)

### Security
12. **Never hardcode secrets** — use variables, environment variables, or vault
13. **Use `sensitive = true`** on all secret variables and outputs
14. **Encrypt state at rest** (KMS for S3, Azure encryption)
15. **Restrict state access** with IAM/RBAC policies
16. **Pin provider versions** to avoid supply-chain attacks
17. **Run tfsec/checkov** in CI/CD pipelines
18. **Use OIDC authentication** instead of static credentials

### Performance & Operations
19. **Use `-target`** sparingly and only for debugging
20. **Use `terraform plan -out=plan.tfplan`** to ensure apply matches plan
21. **Use `moved` blocks** for refactoring instead of state mv
22. **Keep state files small** — split large projects into multiple state files
23. **Use data sources** to reference resources from other state files
24. **Run `terraform fmt`** before every commit
25. **Document modules** with descriptions on all variables and outputs

---

## 12. Common Errors and Troubleshooting

### State Lock Errors

```
Error: Error acquiring the state lock
Lock Info:
  ID:        a1b2c3d4-e5f6-7890-abcd-ef1234567890
  Path:      terraform-state/prod/terraform.tfstate
  Operation: OperationTypeApply
```

**Fix:**
```bash
# If the process that held the lock crashed:
terraform force-unlock a1b2c3d4-e5f6-7890-abcd-ef1234567890

# Verify no one else is running terraform first!
```

### Provider Version Conflicts

```
Error: Failed to query available provider packages
Could not retrieve the list of available versions for provider hashicorp/aws
```

**Fix:**
```bash
# Clear provider cache
rm -rf .terraform/
terraform init -upgrade
```

### Dependency Cycles

```
Error: Cycle: aws_security_group.a, aws_security_group.b
```

**Fix:** Use separate `aws_security_group_rule` resources instead of inline rules:
```hcl
# Instead of inline ingress referencing each other:
resource "aws_security_group" "a" {
  name   = "sg-a"
  vpc_id = aws_vpc.main.id
}

resource "aws_security_group" "b" {
  name   = "sg-b"
  vpc_id = aws_vpc.main.id
}

resource "aws_security_group_rule" "a_from_b" {
  type                     = "ingress"
  from_port                = 443
  to_port                  = 443
  protocol                 = "tcp"
  security_group_id        = aws_security_group.a.id
  source_security_group_id = aws_security_group.b.id
}
```

### Resource Already Exists

```
Error: creating EC2 Instance: InvalidParameterValue: 
  An instance with name 'my-server' already exists
```

**Fix:**
```bash
# Option 1: Import the existing resource
terraform import aws_instance.my_server i-existing123

# Option 2: Remove from state if you want to recreate
terraform state rm aws_instance.my_server
terraform apply
```

### Drift Detection

```bash
# Detect drift
terraform plan -refresh-only

# If drift is found, you have two options:
# 1. Accept the drift (update state to match reality)
terraform apply -refresh-only

# 2. Correct the drift (make reality match config)
terraform apply
```

---

## 13. Terraform CLI Commands (Complete Reference)

```bash
# === INITIALIZATION ===
terraform init                    # Initialize working directory
terraform init -upgrade           # Upgrade providers to latest allowed
terraform init -migrate-state     # Migrate state to new backend
terraform init -reconfigure       # Reconfigure backend, ignore existing

# === PLANNING ===
terraform plan                    # Show execution plan
terraform plan -out=plan.tfplan   # Save plan to file
terraform plan -target=aws_instance.web  # Plan single resource
terraform plan -var="env=prod"    # Pass variable
terraform plan -var-file="prod.tfvars"   # Use variable file
terraform plan -destroy           # Plan destruction
terraform plan -refresh-only      # Only check for drift

# === APPLYING ===
terraform apply                   # Apply changes (interactive)
terraform apply -auto-approve     # Skip confirmation
terraform apply plan.tfplan       # Apply saved plan
terraform apply -target=resource  # Apply single resource
terraform apply -replace=resource # Force recreate resource

# === DESTRUCTION ===
terraform destroy                 # Destroy all resources
terraform destroy -target=resource # Destroy single resource
terraform destroy -auto-approve   # Skip confirmation

# === STATE MANAGEMENT ===
terraform state list              # List resources in state
terraform state show <resource>   # Show resource details
terraform state mv <src> <dst>    # Rename resource in state
terraform state rm <resource>     # Remove from state (don't destroy)
terraform state pull              # Download remote state
terraform state push              # Upload state (dangerous)
terraform force-unlock <id>       # Release stuck lock

# === IMPORT ===
terraform import <resource> <id>  # Import existing infrastructure
terraform plan -generate-config-out=gen.tf  # Generate config for imports

# === WORKSPACE ===
terraform workspace list          # List workspaces
terraform workspace new <name>    # Create workspace
terraform workspace select <name> # Switch workspace
terraform workspace delete <name> # Delete workspace
terraform workspace show          # Current workspace

# === FORMATTING & VALIDATION ===
terraform fmt                     # Format files
terraform fmt -check              # Check formatting (CI)
terraform fmt -recursive          # Format all subdirectories
terraform validate                # Validate configuration syntax

# === INSPECTION ===
terraform output                  # Show all outputs
terraform output <name>           # Show specific output
terraform output -json            # Outputs as JSON
terraform show                    # Show current state
terraform show plan.tfplan        # Show saved plan
terraform graph                   # Generate dependency graph (DOT format)
terraform providers               # Show required providers
terraform version                 # Show Terraform version

# === OTHER ===
terraform console                 # Interactive expression evaluator
terraform taint <resource>        # Mark for recreation (deprecated, use -replace)
terraform untaint <resource>      # Remove taint mark
terraform refresh                 # Update state (deprecated, use plan -refresh-only)
```

---

## 14. Lessons Learned / Pro Tips (20 Tips)

1. **Always run `plan` before `apply`** — never skip this step, even in dev
2. **Use `-out` flag with plan** — guarantees apply does exactly what plan showed
3. **State is sacred** — treat your state file like a database. Back it up, encrypt it, restrict access
4. **Start simple, refactor later** — don't over-engineer modules on day one
5. **Use `moved` blocks for refactoring** — safer than `terraform state mv`
6. **Set up remote state FIRST** — before writing any resources
7. **One state per blast radius** — separate networking, compute, and database states
8. **Use `terraform console`** — excellent for testing expressions and functions
9. **Read the plan output carefully** — `~` means update in-place, `+/-` means destroy and recreate
10. **Use `prevent_destroy` on databases** — one wrong `terraform destroy` can ruin your day
11. **Lock your provider versions** — `~> 5.0` not `>= 5.0` to avoid breaking changes
12. **Use `terraform fmt` pre-commit hook** — consistent formatting across team
13. **Don't use `count` for resources with identity** — use `for_each` instead
14. **Never run `terraform apply` without reviewing the plan** in production
15. **Use `ignore_changes` wisely** — for fields managed by external processes (auto-scaling, etc.)
16. **Tag everything** — makes cost tracking and filtering possible
17. **Use `data` sources to avoid hardcoding** — look up AMIs, AZs, account IDs dynamically
18. **Split large configurations** — if `plan` takes more than 30 seconds, consider splitting
19. **Use `depends_on` as last resort** — Terraform usually infers dependencies correctly
20. **Document your modules** — future you (or your replacement) will thank you

---

## 15. Interview Questions (25 with Detailed Answers)

**Q1: What is Terraform and how does it differ from Ansible?**
> Terraform is an IaC tool for provisioning infrastructure (creating VMs, networks, databases). Ansible is a configuration management tool for configuring servers (installing packages, managing services). Terraform is declarative and manages state; Ansible is procedural and stateless. They complement each other: Terraform creates the VM, Ansible configures it.

**Q2: Explain the Terraform workflow (init, plan, apply).**
> `init` downloads providers and initializes the backend. `plan` compares desired state (config) with current state (state file) and shows what will change. `apply` executes the changes and updates the state file. This workflow ensures you always know what will change before it happens.

**Q3: What is Terraform state and why is it critical?**
> State is a JSON file that maps your configuration to real cloud resources. It stores resource IDs, attributes, and dependencies. Without state, Terraform cannot determine what exists, what needs to change, or what to destroy. It's the source of truth for Terraform's understanding of your infrastructure.

**Q4: How do you handle state in a team environment?**
> Use remote backends (S3+DynamoDB, Azure Blob, Terraform Cloud). Remote state provides: shared access, state locking (prevents concurrent modifications), encryption at rest, and versioning for rollback. Never commit state files to git.

**Q5: What is the difference between `count` and `for_each`?**
> `count` creates numbered instances (index-based). Removing an item from the middle reindexes all subsequent resources, causing unnecessary destroy/recreate. `for_each` creates named instances (key-based). Removing an item only affects that specific resource. Use `for_each` for resources with identity; `count` only for simple numeric scaling or conditional creation (count = condition ? 1 : 0).

**Q6: How do you handle secrets in Terraform?**
> Never hardcode secrets. Options: 1) Use `sensitive = true` on variables, 2) Pass via environment variables (TF_VAR_*), 3) Use AWS Secrets Manager/Azure Key Vault data sources, 4) Use HashiCorp Vault provider, 5) Use CI/CD secrets injection. Note: secrets still appear in state file — always encrypt state.

**Q7: Explain Terraform modules and their benefits.**
> Modules are reusable packages of Terraform configuration. Benefits: DRY (Don't Repeat Yourself), encapsulation (hide complexity), consistency (same patterns everywhere), versioning (pin module versions), and testing (test module in isolation). Structure: variables.tf (inputs), main.tf (resources), outputs.tf (return values).

**Q8: What happens if two people run `terraform apply` simultaneously?**
> With proper backend configuration (state locking), the second person gets a lock error and must wait. Without locking, state corruption is likely — both apply different changes based on the same state, resulting in lost updates and inconsistencies.

**Q9: How do you import existing infrastructure into Terraform?**
> Use `terraform import <resource_address> <resource_id>`. This adds the resource to state. You must also write the corresponding HCL configuration manually. In Terraform 1.5+, use `import` blocks and `terraform plan -generate-config-out=generated.tf` to auto-generate configuration.

**Q10: What is a Terraform provider?**
> A provider is a plugin that interfaces with an API (AWS, Azure, GCP, Kubernetes, etc.). It defines available resources and data sources. Providers handle authentication, API calls, and resource lifecycle. They're downloaded during `terraform init`.

**Q11: Explain `terraform taint` and its replacement.**
> `terraform taint` marks a resource for destruction and recreation on next apply. It's deprecated in favor of `terraform apply -replace=<resource>`, which is more explicit and doesn't require modifying state between plan and apply.

**Q12: What are data sources in Terraform?**
> Data sources read information from existing infrastructure without managing it. Examples: looking up the latest AMI, getting current AWS account ID, reading another state file's outputs. They're read-only and refresh on every plan.

**Q13: How do you manage multiple environments (dev/staging/prod)?**
> Options: 1) Separate directories with shared modules (recommended), 2) Workspaces, 3) Variable files per environment. Best practice: use directory structure with a shared modules directory and per-environment tfvars. This gives maximum isolation and clear access control.

**Q14: What is `terraform plan -refresh-only`?**
> It compares the state file against real infrastructure to detect drift (changes made outside Terraform). It doesn't compare against your configuration. Used to discover if someone manually modified resources.

**Q15: Explain lifecycle meta-arguments.**
> `create_before_destroy`: creates replacement before destroying original (zero-downtime). `prevent_destroy`: blocks `terraform destroy` on the resource. `ignore_changes`: ignores specified attribute changes (for externally managed fields). `replace_triggered_by`: forces recreation when another resource changes.

**Q16: What is a Terraform backend?**
> A backend defines where state is stored and how operations are executed. Local backend stores state on disk. Remote backends (S3, Azure Blob, Terraform Cloud) enable team collaboration, locking, and encryption. Configured in the `terraform {}` block.

**Q17: How do you handle Terraform in a CI/CD pipeline?**
> Plan on PR (comment results), apply on merge to main (with manual approval for prod). Use saved plan files (-out flag) to ensure apply matches plan. Store credentials as CI/CD secrets. Use OIDC for cloud auth. Run fmt/validate/tfsec as checks.

**Q18: What is the `depends_on` meta-argument?**
> `depends_on` explicitly declares a dependency that Terraform can't automatically detect. Use sparingly — Terraform infers most dependencies from resource references. Only needed when there's a hidden dependency (e.g., IAM policy must exist before resource that uses it, but there's no direct reference).

**Q19: Explain the difference between `variable` and `local`.**
> Variables are inputs — set by the user via tfvars, CLI flags, or env vars. Locals are computed values — derived from variables, expressions, or other locals. Variables create the module's API; locals are internal computed helpers.

**Q20: What is `terraform.tfvars` and how is it used?**
> It's an auto-loaded file containing variable values. Terraform automatically loads `terraform.tfvars` and `*.auto.tfvars`. Other files need explicit `-var-file` flag. Used to provide environment-specific values without changing code.

**Q21: How do you handle resource dependencies across state files?**
> Use `terraform_remote_state` data source to read outputs from another state file, or use data sources to query resources directly. Example: network team manages VPC (state A), app team references VPC ID via remote state (state B).

**Q22: What are `moved` blocks?**
> Introduced in Terraform 1.1, `moved` blocks declare that a resource has been renamed/refactored. They tell Terraform to update state instead of destroying and recreating. Safer than `terraform state mv` because they're version-controlled and team-friendly.

**Q23: How do you debug Terraform issues?**
> Set `TF_LOG=DEBUG` environment variable for detailed logs. Use `terraform console` to test expressions. Use `terraform state show` to inspect resources. Use `terraform graph` for dependency visualization. Check `.terraform` directory for provider issues.

**Q24: What are Terraform provisioners and why should you avoid them?**
> Provisioners (local-exec, remote-exec, file) run scripts during resource creation. Avoid because: they break the declarative model, aren't captured in state, can't be planned, and create fragile dependencies. Use cloud-init, Packer, or Ansible instead.

**Q25: Explain the Terraform provider version constraint `~> 5.0`.**
> The `~>` (pessimistic constraint) allows the rightmost version component to increment. `~> 5.0` means `>= 5.0, < 6.0` (any 5.x). `~> 5.1` means `>= 5.1, < 5.2`. This balances getting patches while avoiding breaking changes.

---

## 16. Free Resources

1. **HashiCorp Learn** — https://developer.hashicorp.com/terraform/tutorials — Official tutorials from beginner to advanced
2. **Terraform Documentation** — https://developer.hashicorp.com/terraform/docs — Complete reference
3. **Terraform Registry** — https://registry.terraform.io — Providers and modules
4. **Terraform Best Practices** — https://www.terraform-best-practices.com — Community guide
5. **tfsec** — https://github.com/aquasecurity/tfsec — Static analysis for security
6. **Checkov** — https://www.checkov.io — Policy-as-code scanner
7. **Spacelift Blog** — https://spacelift.io/blog — Excellent Terraform articles
8. **Gruntwork Blog** — https://blog.gruntwork.io — Production Terraform patterns
9. **CloudPosse Modules** — https://github.com/cloudposse — Open-source production modules
10. **Awesome Terraform** — https://github.com/shuaibiyy/awesome-terraform — Curated resource list
11. **KodeKloud Terraform Course** — https://kodekloud.com — Hands-on labs (free tier available)
12. **freeCodeCamp Terraform YouTube** — https://www.youtube.com/watch?v=SLB_c_ayRMo — Full course

---

## 17. HashiCorp Terraform Associate Certification Prep

### Exam Details
- **Code:** HashiCorp Certified: Terraform Associate (003)
- **Duration:** 60 minutes
- **Questions:** ~57 multiple choice/multi-select
- **Passing Score:** ~70%
- **Cost:** $70.50 USD
- **Validity:** 2 years

### Exam Objectives (What to Study)

1. **Understand IaC concepts** (declarative vs imperative, benefits)
2. **Understand Terraform's purpose** (multi-cloud, provider ecosystem)
3. **Understand Terraform basics** (providers, init/plan/apply, plugin-based arch)
4. **Use Terraform CLI** (all commands, workspace, fmt, validate)
5. **Interact with Terraform modules** (sources, inputs/outputs, versioning)
6. **Navigate Terraform workflow** (write, plan, apply, destroy)
7. **Implement and maintain state** (backends, locking, state commands)
8. **Read, generate, and modify config** (HCL, variables, outputs, resources, data sources)
9. **Understand Terraform Cloud/Enterprise** (remote execution, sentinel, workspaces)

### Study Tips
- **Practice hands-on** — use the free tier of AWS/Azure to create real infrastructure
- **Focus on state management** — 20-25% of exam questions relate to state
- **Know ALL CLI commands** — especially less common ones like `taint`, `import`, `console`
- **Understand backends** — how to configure, migrate, and troubleshoot
- **Practice with variables** — type constraints, validation, precedence order
- **Know module sources** — local, registry, git, S3, etc.
- **Terraform Cloud questions** — understand workspaces, runs, sentinel basics
- **Variable precedence** (highest to lowest): CLI `-var`, `*.auto.tfvars`, `terraform.tfvars`, env vars `TF_VAR_*`, defaults

### Sample Practice Workflow
```bash
# 1. Create a project structure
mkdir -p terraform-practice/{modules,environments/{dev,prod}}

# 2. Write config, plan, apply, modify, plan, apply
# 3. Practice imports, state commands, workspace operations
# 4. Break things intentionally and fix them
# 5. Time yourself — 57 questions in 60 minutes = ~1 min per question
```

---

## Quick Reference Card

```
terraform init          → Download plugins, init backend
terraform plan          → Preview changes
terraform apply         → Execute changes
terraform destroy       → Remove everything
terraform fmt           → Format code
terraform validate      → Check syntax
terraform state list    → Show managed resources
terraform import        → Bring existing resource under management
terraform output        → Show output values
terraform console       → Interactive REPL
```

---

*Last updated: July 2026 | Created for the DevOps Upskilling Program*
*Next: Month 3 Part 2 — Terraform Advanced Patterns, Terragrunt, and Policy as Code*
