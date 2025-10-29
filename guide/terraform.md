# Terraform Beginner's Guide

## Table of Contents
1. [What is Terraform?](#what-is-terraform)
2. [Infrastructure as Code (IaC)](#infrastructure-as-code-iac)
3. [Terraform Installation](#terraform-installation)
4. [Terraform Basics](#terraform-basics)
5. [Terraform Configuration Language (HCL)](#terraform-configuration-language-hcl)
6. [Providers and Resources](#providers-and-resources)
7. [Variables and Outputs](#variables-and-outputs)
8. [State Management](#state-management)
9. [Modules](#modules)
10. [AWS with Terraform](#aws-with-terraform)
11. [Best Practices](#best-practices)
12. [Advanced Topics](#advanced-topics)

## What is Terraform?

Terraform is an open-source Infrastructure as Code (IaC) tool created by HashiCorp that allows you to define, provision, and manage infrastructure using declarative configuration files. Think of it as a way to describe your infrastructure using code rather than clicking through web consoles.

### Key Features

- **Infrastructure as Code**: Define infrastructure in human-readable configuration files
- **Multi-Cloud**: Works with AWS, Azure, GCP, and 1000+ providers
- **Declarative**: Describe what you want, not how to do it
- **State Management**: Tracks your infrastructure's current state
- **Plan and Apply**: Preview changes before applying them
- **Resource Graph**: Understands dependencies between resources

### Benefits of Terraform

- **Consistency**: Same infrastructure every time
- **Version Control**: Track changes and collaborate
- **Automation**: Integrate with CI/CD pipelines
- **Cost Management**: Easily tear down unused resources
- **Documentation**: Infrastructure defined as code serves as documentation
- **Scalability**: Manage infrastructure at any scale

### Terraform vs Other Tools

| Tool | Type | Language | Cloud Support |
|------|------|----------|---------------|
| Terraform | Declarative | HCL | Multi-cloud |
| CloudFormation | Declarative | JSON/YAML | AWS only |
| ARM Templates | Declarative | JSON | Azure only |
| Pulumi | Imperative/Declarative | Multiple languages | Multi-cloud |
| Ansible | Imperative | YAML | Configuration management |

## Infrastructure as Code (IaC)

### What is IaC?

Infrastructure as Code is the practice of managing and provisioning infrastructure through machine-readable definition files, rather than physical hardware configuration or interactive configuration tools.

### Benefits of IaC

**Consistency**
- Eliminates configuration drift
- Same infrastructure across environments
- Reduces human error

**Speed**
- Rapid provisioning and deployment
- Automated infrastructure setup
- Faster time to market

**Cost Control**
- Easy to tear down unused resources
- Track resource usage
- Optimize costs through code

**Risk Reduction**
- Version controlled infrastructure
- Peer review of changes
- Rollback capabilities

### IaC Principles

1. **Immutable Infrastructure**: Replace rather than modify
2. **Version Control**: All infrastructure code is versioned
3. **Declarative**: Define the desired end state
4. **Idempotent**: Same result regardless of how many times executed
5. **Self-Documenting**: Code serves as documentation

### Traditional vs IaC Approach

**Traditional Approach:**
```
1. Log into AWS Console
2. Click "Launch Instance"
3. Select AMI, instance type, VPC, etc.
4. Configure security groups manually
5. Repeat for each environment
6. Document configuration in wiki
7. Hope nothing changes without documentation
```

**IaC Approach:**
```
1. Write Terraform configuration
2. Version control the configuration
3. terraform plan (preview changes)
4. terraform apply (create infrastructure)
5. Repeat for any environment
6. Infrastructure is self-documenting
7. Track all changes through Git
```

## Terraform Installation

### Installing Terraform

**Windows (using Chocolatey):**
```powershell
# Install Chocolatey first if not installed
Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))

# Install Terraform
choco install terraform

# Verify installation
terraform version
```

**Windows (Manual):**
1. Download from [terraform.io](https://www.terraform.io/downloads.html)
2. Extract the executable to a directory in your PATH
3. Verify: `terraform version`

**Linux (Ubuntu/Debian):**
```bash
# Add HashiCorp GPG key
wget -O- https://apt.releases.hashicorp.com/gpg | gpg --dearmor | sudo tee /usr/share/keyrings/hashicorp-archive-keyring.gpg

# Add HashiCorp repository
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list

# Update and install
sudo apt update && sudo apt install terraform

# Verify installation
terraform version
```

**macOS (using Homebrew):**
```bash
# Install Homebrew if not installed
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Install Terraform
brew tap hashicorp/tap
brew install hashicorp/tap/terraform

# Verify installation
terraform version
```

### Setting up AWS Credentials

**Option 1: AWS CLI Configuration**
```bash
# Install AWS CLI first
# Configure credentials
aws configure

# Verify
aws sts get-caller-identity
```

**Option 2: Environment Variables**
```bash
# Windows (PowerShell)
$env:AWS_ACCESS_KEY_ID="your-access-key"
$env:AWS_SECRET_ACCESS_KEY="your-secret-key"
$env:AWS_DEFAULT_REGION="us-west-2"

# Linux/macOS
export AWS_ACCESS_KEY_ID="your-access-key"
export AWS_SECRET_ACCESS_KEY="your-secret-key"
export AWS_DEFAULT_REGION="us-west-2"
```

**Option 3: IAM Roles (recommended for EC2)**
- Attach IAM role to EC2 instance
- No credentials needed in code
- More secure approach

## Terraform Basics

### Terraform Workflow

1. **Write**: Define infrastructure in `.tf` files
2. **Plan**: Preview changes with `terraform plan`
3. **Apply**: Create infrastructure with `terraform apply`
4. **Manage**: Modify configuration and repeat

### Core Commands

```bash
# Initialize Terraform (run once per project)
terraform init

# Validate configuration syntax
terraform validate

# Format configuration files
terraform fmt

# Plan changes (dry run)
terraform plan

# Apply changes
terraform apply

# Show current state
terraform show

# List resources in state
terraform state list

# Destroy infrastructure
terraform destroy
```

### Your First Terraform Configuration

Create a file named `main.tf`:

```hcl
# Configure the AWS Provider
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
  required_version = ">= 1.0"
}

provider "aws" {
  region = "us-west-2"
}

# Create a VPC
resource "aws_vpc" "main" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_hostnames = true
  enable_dns_support   = true

  tags = {
    Name = "main-vpc"
  }
}

# Create an Internet Gateway
resource "aws_internet_gateway" "main" {
  vpc_id = aws_vpc.main.id

  tags = {
    Name = "main-igw"
  }
}

# Create a public subnet
resource "aws_subnet" "public" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.1.0/24"
  availability_zone       = "us-west-2a"
  map_public_ip_on_launch = true

  tags = {
    Name = "public-subnet"
  }
}
```

**Run your first Terraform commands:**
```bash
# Initialize
terraform init

# Plan
terraform plan

# Apply
terraform apply
```

## Terraform Configuration Language (HCL)

### HCL Syntax Basics

```hcl
# Comments start with #

# Block syntax
resource_type "resource_name" {
  argument1 = value1
  argument2 = value2
  
  nested_block {
    nested_argument = "value"
  }
}

# String values
variable "example_string" {
  default = "Hello, World!"
}

# Number values
variable "example_number" {
  default = 42
}

# Boolean values
variable "example_bool" {
  default = true
}

# List values
variable "example_list" {
  default = ["item1", "item2", "item3"]
}

# Map values
variable "example_map" {
  default = {
    key1 = "value1"
    key2 = "value2"
  }
}
```

### Data Types

**Primitive Types:**
- `string`: Text values
- `number`: Numeric values
- `bool`: true or false

**Collection Types:**
- `list(type)`: Ordered sequence of values
- `set(type)`: Unordered collection of unique values
- `map(type)`: Key-value pairs

**Structural Types:**
- `object({...})`: Collection of named attributes
- `tuple([...])`: Sequence of elements of different types

### Expressions and Functions

```hcl
# String interpolation
name = "server-${var.environment}"

# Conditional expressions
instance_type = var.environment == "production" ? "t3.large" : "t3.micro"

# Function calls
vpc_cidr = cidrsubnet("10.0.0.0/16", 8, 1)
timestamp = formatdate("YYYY-MM-DD", timestamp())

# For expressions
subnet_ids = [for subnet in aws_subnet.private : subnet.id]

# Common functions
upper("hello")           # "HELLO"
lower("WORLD")           # "world"
length([1, 2, 3])        # 3
max(1, 2, 3)            # 3
min(1, 2, 3)            # 1
join(",", ["a", "b"])    # "a,b"
split(",", "a,b,c")      # ["a", "b", "c"]
```

### Local Values

```hcl
locals {
  common_tags = {
    Project     = "MyProject"
    Environment = var.environment
    Owner       = "DevOps Team"
  }
  
  vpc_cidr = "10.0.0.0/16"
  
  availability_zones = [
    "${var.region}a",
    "${var.region}b",
    "${var.region}c"
  ]
}

resource "aws_instance" "web" {
  ami           = var.ami_id
  instance_type = var.instance_type
  
  tags = merge(local.common_tags, {
    Name = "web-server"
  })
}
```

## Providers and Resources

### Providers

Providers are plugins that Terraform uses to manage resources. They're responsible for understanding API interactions and exposing resources.

```hcl
# AWS Provider
provider "aws" {
  region  = "us-west-2"
  profile = "default"
  
  default_tags {
    tags = {
      Project = "MyProject"
      Owner   = "DevOps"
    }
  }
}

# Azure Provider
provider "azurerm" {
  features {}
}

# Google Cloud Provider
provider "google" {
  project = "my-project-id"
  region  = "us-central1"
}

# Multiple Provider Instances
provider "aws" {
  alias  = "west"
  region = "us-west-2"
}

provider "aws" {
  alias  = "east"
  region = "us-east-1"
}

# Using specific provider
resource "aws_instance" "west_server" {
  provider = aws.west
  # ... other configuration
}
```

### Resources

Resources are the most important element in Terraform. They describe infrastructure objects.

```hcl
# Basic resource syntax
resource "resource_type" "resource_name" {
  argument1 = value1
  argument2 = value2
}

# AWS EC2 Instance
resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1d0"
  instance_type = "t3.micro"
  key_name      = "my-key"
  
  vpc_security_group_ids = [aws_security_group.web.id]
  subnet_id              = aws_subnet.public.id
  
  user_data = <<-EOF
    #!/bin/bash
    yum update -y
    yum install -y httpd
    systemctl start httpd
    systemctl enable httpd
    echo "<h1>Hello from Terraform!</h1>" > /var/www/html/index.html
  EOF
  
  tags = {
    Name = "web-server"
  }
}

# AWS Security Group
resource "aws_security_group" "web" {
  name_prefix = "web-"
  vpc_id      = aws_vpc.main.id

  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["10.0.0.0/16"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}
```

### Data Sources

Data sources allow Terraform to use information defined outside of Terraform or by another Terraform configuration.

```hcl
# Get latest Amazon Linux 2 AMI
data "aws_ami" "amazon_linux" {
  most_recent = true
  owners      = ["amazon"]
  
  filter {
    name   = "name"
    values = ["amzn2-ami-hvm-*-x86_64-gp2"]
  }
  
  filter {
    name   = "virtualization-type"
    values = ["hvm"]
  }
}

# Get current AWS region
data "aws_region" "current" {}

# Get current AWS caller identity
data "aws_caller_identity" "current" {}

# Get availability zones
data "aws_availability_zones" "available" {
  state = "available"
}

# Use data source in resource
resource "aws_instance" "web" {
  ami           = data.aws_ami.amazon_linux.id
  instance_type = "t3.micro"
  
  availability_zone = data.aws_availability_zones.available.names[0]
  
  tags = {
    Name      = "web-server"
    Region    = data.aws_region.current.name
    AccountId = data.aws_caller_identity.current.account_id
  }
}
```

### Resource Dependencies

```hcl
# Implicit dependency (recommended)
resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1d0"
  instance_type = "t3.micro"
  subnet_id     = aws_subnet.public.id  # Implicit dependency
}

# Explicit dependency (when implicit isn't enough)
resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1d0"
  instance_type = "t3.micro"
  
  depends_on = [aws_internet_gateway.main]
}
```

## Variables and Outputs

### Input Variables

Variables allow you to parameterize your Terraform configurations.

```hcl
# Variable declarations
variable "instance_type" {
  description = "EC2 instance type"
  type        = string
  default     = "t3.micro"
}

variable "environment" {
  description = "Environment name"
  type        = string
  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "Environment must be dev, staging, or prod."
  }
}

variable "availability_zones" {
  description = "List of availability zones"
  type        = list(string)
  default     = ["us-west-2a", "us-west-2b"]
}

variable "tags" {
  description = "Common tags for all resources"
  type        = map(string)
  default = {
    Project = "MyProject"
    Owner   = "DevOps"
  }
}

# Using variables
resource "aws_instance" "web" {
  ami           = data.aws_ami.amazon_linux.id
  instance_type = var.instance_type
  
  tags = merge(var.tags, {
    Name        = "web-${var.environment}"
    Environment = var.environment
  })
}
```

### Variable Files

**terraform.tfvars:**
```hcl
instance_type = "t3.medium"
environment   = "production"
availability_zones = [
  "us-west-2a",
  "us-west-2b",
  "us-west-2c"
]
tags = {
  Project     = "WebApp"
  Owner       = "WebTeam"
  Environment = "production"
}
```

**dev.tfvars:**
```hcl
instance_type = "t3.micro"
environment   = "dev"
```

**Using variable files:**
```bash
# Use default terraform.tfvars
terraform apply

# Use specific variable file
terraform apply -var-file="dev.tfvars"

# Override specific variables
terraform apply -var="instance_type=t3.large"
```

### Output Values

Outputs make information about your infrastructure available on the command line and to other Terraform configurations.

```hcl
# Output values
output "vpc_id" {
  description = "ID of the VPC"
  value       = aws_vpc.main.id
}

output "instance_ip" {
  description = "Public IP address of the instance"
  value       = aws_instance.web.public_ip
}

output "instance_dns" {
  description = "Public DNS name of the instance"
  value       = aws_instance.web.public_dns
}

output "database_endpoint" {
  description = "RDS instance endpoint"
  value       = aws_db_instance.main.endpoint
  sensitive   = true  # Mark as sensitive (won't display in logs)
}

# Conditional output
output "load_balancer_dns" {
  description = "DNS name of the load balancer"
  value       = var.create_load_balancer ? aws_lb.main[0].dns_name : null
}
```

**Viewing outputs:**
```bash
# Show all outputs
terraform output

# Show specific output
terraform output vpc_id

# Show output in JSON format
terraform output -json

# Show sensitive outputs
terraform output -json | jq '.database_endpoint.value'
```

## State Management

### Understanding Terraform State

Terraform state is a JSON file that maps your configuration to real-world resources. It's crucial for Terraform to know what it has created.

**State file (terraform.tfstate) contains:**
- Resource mappings
- Metadata
- Dependencies
- Provider configuration

### Local State vs Remote State

**Local State:**
- Stored in `terraform.tfstate` file
- Good for learning and testing
- Not suitable for teams
- Risk of loss or corruption

**Remote State:**
- Stored in remote backend (S3, Consul, etc.)
- Supports team collaboration
- Provides locking
- More secure and reliable

### Remote State with S3

```hcl
# backend.tf
terraform {
  backend "s3" {
    bucket         = "my-terraform-state-bucket"
    key            = "infrastructure/terraform.tfstate"
    region         = "us-west-2"
    encrypt        = true
    dynamodb_table = "terraform-locks"
  }
}
```

**Setting up S3 backend:**

1. **Create S3 bucket:**
```bash
aws s3 mb s3://my-terraform-state-bucket --region us-west-2

# Enable versioning
aws s3api put-bucket-versioning \
  --bucket my-terraform-state-bucket \
  --versioning-configuration Status=Enabled

# Enable encryption
aws s3api put-bucket-encryption \
  --bucket my-terraform-state-bucket \
  --server-side-encryption-configuration '{
    "Rules": [
      {
        "ApplyServerSideEncryptionByDefault": {
          "SSEAlgorithm": "AES256"
        }
      }
    ]
  }'
```

2. **Create DynamoDB table for locking:**
```bash
aws dynamodb create-table \
  --table-name terraform-locks \
  --attribute-definitions AttributeName=LockID,AttributeType=S \
  --key-schema AttributeName=LockID,KeyType=HASH \
  --provisioned-throughput ReadCapacityUnits=5,WriteCapacityUnits=5
```

3. **Initialize backend:**
```bash
terraform init
```

### State Commands

```bash
# List resources in state
terraform state list

# Show resource details
terraform state show aws_instance.web

# Move resource in state
terraform state mv aws_instance.web aws_instance.web_server

# Remove resource from state (doesn't destroy)
terraform state rm aws_instance.web

# Import existing resource into state
terraform import aws_instance.web i-1234567890abcdef0

# Refresh state from real infrastructure
terraform refresh

# Replace corrupted resource
terraform apply -replace=aws_instance.web
```

### State Locking

State locking prevents multiple users from running Terraform simultaneously against the same state.

```hcl
terraform {
  backend "s3" {
    bucket         = "my-terraform-state-bucket"
    key            = "infrastructure/terraform.tfstate"
    region         = "us-west-2"
    encrypt        = true
    dynamodb_table = "terraform-locks"  # Enables locking
  }
}
```

## Modules

### What are Modules?

Modules are reusable Terraform configurations that can be called multiple times with different inputs.

**Benefits:**
- Code reusability
- Consistent infrastructure patterns
- Easier maintenance
- Better organization

### Creating a Module

**Directory structure:**
```
modules/
└── web-server/
    ├── main.tf
    ├── variables.tf
    ├── outputs.tf
    └── README.md
```

**modules/web-server/variables.tf:**
```hcl
variable "instance_type" {
  description = "EC2 instance type"
  type        = string
  default     = "t3.micro"
}

variable "vpc_id" {
  description = "VPC ID where resources will be created"
  type        = string
}

variable "subnet_id" {
  description = "Subnet ID for the instance"
  type        = string
}

variable "name" {
  description = "Name tag for the instance"
  type        = string
}

variable "allowed_cidr_blocks" {
  description = "CIDR blocks allowed to access the instance"
  type        = list(string)
  default     = ["0.0.0.0/0"]
}
```

**modules/web-server/main.tf:**
```hcl
data "aws_ami" "amazon_linux" {
  most_recent = true
  owners      = ["amazon"]
  
  filter {
    name   = "name"
    values = ["amzn2-ami-hvm-*-x86_64-gp2"]
  }
}

resource "aws_security_group" "web" {
  name_prefix = "${var.name}-"
  vpc_id      = var.vpc_id

  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = var.allowed_cidr_blocks
  }

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = var.allowed_cidr_blocks
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "${var.name}-sg"
  }
}

resource "aws_instance" "web" {
  ami           = data.aws_ami.amazon_linux.id
  instance_type = var.instance_type
  subnet_id     = var.subnet_id
  
  vpc_security_group_ids = [aws_security_group.web.id]
  
  user_data = <<-EOF
    #!/bin/bash
    yum update -y
    yum install -y httpd
    systemctl start httpd
    systemctl enable httpd
    echo "<h1>Hello from ${var.name}!</h1>" > /var/www/html/index.html
  EOF
  
  tags = {
    Name = var.name
  }
}
```

**modules/web-server/outputs.tf:**
```hcl
output "instance_id" {
  description = "ID of the EC2 instance"
  value       = aws_instance.web.id
}

output "public_ip" {
  description = "Public IP of the instance"
  value       = aws_instance.web.public_ip
}

output "security_group_id" {
  description = "ID of the security group"
  value       = aws_security_group.web.id
}
```

### Using Modules

**main.tf:**
```hcl
# Use local module
module "web_server_1" {
  source = "./modules/web-server"
  
  name                = "web-server-1"
  instance_type       = "t3.micro"
  vpc_id             = aws_vpc.main.id
  subnet_id          = aws_subnet.public.id
  allowed_cidr_blocks = ["10.0.0.0/16"]
}

module "web_server_2" {
  source = "./modules/web-server"
  
  name                = "web-server-2"
  instance_type       = "t3.small"
  vpc_id             = aws_vpc.main.id
  subnet_id          = aws_subnet.public.id
  allowed_cidr_blocks = ["10.0.0.0/16"]
}

# Use remote module from Terraform Registry
module "vpc" {
  source = "terraform-aws-modules/vpc/aws"
  version = "3.14.0"

  name = "my-vpc"
  cidr = "10.0.0.0/16"

  azs             = ["us-west-2a", "us-west-2b", "us-west-2c"]
  private_subnets = ["10.0.1.0/24", "10.0.2.0/24", "10.0.3.0/24"]
  public_subnets  = ["10.0.101.0/24", "10.0.102.0/24", "10.0.103.0/24"]

  enable_nat_gateway = true
  enable_vpn_gateway = true

  tags = {
    Terraform = "true"
    Environment = "dev"
  }
}
```

**Access module outputs:**
```hcl
output "web_server_1_ip" {
  value = module.web_server_1.public_ip
}

output "vpc_id" {
  value = module.vpc.vpc_id
}
```

### Module Sources

```hcl
# Local path
module "web_server" {
  source = "./modules/web-server"
}

# Git repository
module "web_server" {
  source = "git::https://github.com/username/terraform-modules.git//web-server"
}

# Git with specific branch/tag
module "web_server" {
  source = "git::https://github.com/username/terraform-modules.git//web-server?ref=v1.0.0"
}

# Terraform Registry
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "3.14.0"
}

# HTTP URL
module "web_server" {
  source = "https://example.com/modules/web-server.zip"
}
```

## AWS with Terraform

### Complete AWS Infrastructure Example

```hcl
# variables.tf
variable "region" {
  description = "AWS region"
  type        = string
  default     = "us-west-2"
}

variable "environment" {
  description = "Environment name"
  type        = string
  default     = "dev"
}

variable "project" {
  description = "Project name"
  type        = string
  default     = "myproject"
}
```

**main.tf:**
```hcl
terraform {
  required_version = ">= 1.0"
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = var.region
  
  default_tags {
    tags = {
      Project     = var.project
      Environment = var.environment
      ManagedBy   = "Terraform"
    }
  }
}

# Data sources
data "aws_availability_zones" "available" {
  state = "available"
}

data "aws_ami" "amazon_linux" {
  most_recent = true
  owners      = ["amazon"]
  
  filter {
    name   = "name"
    values = ["amzn2-ami-hvm-*-x86_64-gp2"]
  }
}

# VPC
resource "aws_vpc" "main" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_hostnames = true
  enable_dns_support   = true

  tags = {
    Name = "${var.project}-vpc"
  }
}

# Internet Gateway
resource "aws_internet_gateway" "main" {
  vpc_id = aws_vpc.main.id

  tags = {
    Name = "${var.project}-igw"
  }
}

# Public Subnets
resource "aws_subnet" "public" {
  count = 2
  
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.${count.index + 1}.0/24"
  availability_zone       = data.aws_availability_zones.available.names[count.index]
  map_public_ip_on_launch = true

  tags = {
    Name = "${var.project}-public-subnet-${count.index + 1}"
    Type = "public"
  }
}

# Private Subnets
resource "aws_subnet" "private" {
  count = 2
  
  vpc_id            = aws_vpc.main.id
  cidr_block        = "10.0.${count.index + 10}.0/24"
  availability_zone = data.aws_availability_zones.available.names[count.index]

  tags = {
    Name = "${var.project}-private-subnet-${count.index + 1}"
    Type = "private"
  }
}

# Elastic IP for NAT Gateway
resource "aws_eip" "nat" {
  count = 2
  
  domain = "vpc"
  depends_on = [aws_internet_gateway.main]

  tags = {
    Name = "${var.project}-eip-${count.index + 1}"
  }
}

# NAT Gateway
resource "aws_nat_gateway" "main" {
  count = 2
  
  allocation_id = aws_eip.nat[count.index].id
  subnet_id     = aws_subnet.public[count.index].id

  tags = {
    Name = "${var.project}-nat-${count.index + 1}"
  }
}

# Route Tables
resource "aws_route_table" "public" {
  vpc_id = aws_vpc.main.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.main.id
  }

  tags = {
    Name = "${var.project}-public-rt"
  }
}

resource "aws_route_table" "private" {
  count = 2
  
  vpc_id = aws_vpc.main.id

  route {
    cidr_block     = "0.0.0.0/0"
    nat_gateway_id = aws_nat_gateway.main[count.index].id
  }

  tags = {
    Name = "${var.project}-private-rt-${count.index + 1}"
  }
}

# Route Table Associations
resource "aws_route_table_association" "public" {
  count = 2
  
  subnet_id      = aws_subnet.public[count.index].id
  route_table_id = aws_route_table.public.id
}

resource "aws_route_table_association" "private" {
  count = 2
  
  subnet_id      = aws_subnet.private[count.index].id
  route_table_id = aws_route_table.private[count.index].id
}

# Security Groups
resource "aws_security_group" "web" {
  name_prefix = "${var.project}-web-"
  vpc_id      = aws_vpc.main.id

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

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "${var.project}-web-sg"
  }
}

resource "aws_security_group" "app" {
  name_prefix = "${var.project}-app-"
  vpc_id      = aws_vpc.main.id

  ingress {
    from_port       = 8080
    to_port         = 8080
    protocol        = "tcp"
    security_groups = [aws_security_group.web.id]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "${var.project}-app-sg"
  }
}

# Launch Template
resource "aws_launch_template" "web" {
  name_prefix   = "${var.project}-web-"
  image_id      = data.aws_ami.amazon_linux.id
  instance_type = "t3.micro"
  
  vpc_security_group_ids = [aws_security_group.web.id]
  
  user_data = base64encode(<<-EOF
    #!/bin/bash
    yum update -y
    yum install -y httpd
    systemctl start httpd
    systemctl enable httpd
    echo "<h1>Hello from ${var.project}!</h1>" > /var/www/html/index.html
  EOF
  )

  tag_specifications {
    resource_type = "instance"
    tags = {
      Name = "${var.project}-web-server"
    }
  }
}

# Auto Scaling Group
resource "aws_autoscaling_group" "web" {
  name                = "${var.project}-web-asg"
  vpc_zone_identifier = aws_subnet.public[*].id
  target_group_arns   = [aws_lb_target_group.web.arn]
  health_check_type   = "ELB"
  min_size            = 2
  max_size            = 6
  desired_capacity    = 2

  launch_template {
    id      = aws_launch_template.web.id
    version = "$Latest"
  }

  tag {
    key                 = "Name"
    value               = "${var.project}-web-asg"
    propagate_at_launch = false
  }
}

# Application Load Balancer
resource "aws_lb" "web" {
  name               = "${var.project}-web-alb"
  internal           = false
  load_balancer_type = "application"
  security_groups    = [aws_security_group.web.id]
  subnets            = aws_subnet.public[*].id

  enable_deletion_protection = false

  tags = {
    Name = "${var.project}-web-alb"
  }
}

# Target Group
resource "aws_lb_target_group" "web" {
  name     = "${var.project}-web-tg"
  port     = 80
  protocol = "HTTP"
  vpc_id   = aws_vpc.main.id

  health_check {
    enabled             = true
    healthy_threshold   = 2
    interval            = 30
    matcher             = "200"
    path                = "/"
    port                = "traffic-port"
    protocol            = "HTTP"
    timeout             = 5
    unhealthy_threshold = 2
  }

  tags = {
    Name = "${var.project}-web-tg"
  }
}

# Load Balancer Listener
resource "aws_lb_listener" "web" {
  load_balancer_arn = aws_lb.web.arn
  port              = "80"
  protocol          = "HTTP"

  default_action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.web.arn
  }
}

# RDS Subnet Group
resource "aws_db_subnet_group" "main" {
  name       = "${var.project}-db-subnet-group"
  subnet_ids = aws_subnet.private[*].id

  tags = {
    Name = "${var.project}-db-subnet-group"
  }
}

# RDS Security Group
resource "aws_security_group" "rds" {
  name_prefix = "${var.project}-rds-"
  vpc_id      = aws_vpc.main.id

  ingress {
    from_port       = 3306
    to_port         = 3306
    protocol        = "tcp"
    security_groups = [aws_security_group.app.id]
  }

  tags = {
    Name = "${var.project}-rds-sg"
  }
}

# RDS Instance
resource "aws_db_instance" "main" {
  identifier = "${var.project}-db"
  
  engine         = "mysql"
  engine_version = "8.0"
  instance_class = "db.t3.micro"
  
  allocated_storage     = 20
  max_allocated_storage = 100
  storage_type          = "gp2"
  storage_encrypted     = true
  
  db_name  = "myapp"
  username = "admin"
  password = "changeme123!"  # Use AWS Secrets Manager in production
  
  vpc_security_group_ids = [aws_security_group.rds.id]
  db_subnet_group_name   = aws_db_subnet_group.main.name
  
  backup_retention_period = 7
  backup_window          = "03:00-04:00"
  maintenance_window     = "sun:04:00-sun:05:00"
  
  skip_final_snapshot = true
  deletion_protection = false

  tags = {
    Name = "${var.project}-db"
  }
}
```

**outputs.tf:**
```hcl
output "vpc_id" {
  description = "ID of the VPC"
  value       = aws_vpc.main.id
}

output "load_balancer_dns" {
  description = "DNS name of the load balancer"
  value       = aws_lb.web.dns_name
}

output "database_endpoint" {
  description = "RDS instance endpoint"
  value       = aws_db_instance.main.endpoint
  sensitive   = true
}

output "public_subnet_ids" {
  description = "IDs of the public subnets"
  value       = aws_subnet.public[*].id
}

output "private_subnet_ids" {
  description = "IDs of the private subnets"
  value       = aws_subnet.private[*].id
}
```

## Best Practices

### Code Organization

**Directory Structure:**
```
terraform-project/
├── environments/
│   ├── dev/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   ├── outputs.tf
│   │   └── terraform.tfvars
│   ├── staging/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   ├── outputs.tf
│   │   └── terraform.tfvars
│   └── prod/
│       ├── main.tf
│       ├── variables.tf
│       ├── outputs.tf
│       └── terraform.tfvars
├── modules/
│   ├── vpc/
│   ├── web-server/
│   └── database/
├── scripts/
│   ├── deploy.sh
│   └── validate.sh
└── README.md
```

### Naming Conventions

```hcl
# Resource naming
resource "aws_instance" "web_server" {  # snake_case for resource names
  tags = {
    Name = "my-project-web-server"  # kebab-case for AWS resource names
  }
}

# Variable naming
variable "instance_type" {      # snake_case
variable "vpc_cidr_block" {     # descriptive names
variable "enable_monitoring" {   # boolean variables start with "enable" or "is"
```

### Version Constraints

```hcl
terraform {
  required_version = ">= 1.0"
  
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"  # Allow 5.x but not 6.x
    }
    random = {
      source  = "hashicorp/random"
      version = ">= 3.1"  # At least 3.1
    }
  }
}
```

### Security Best Practices

**1. Use Variables for Sensitive Data**
```hcl
variable "db_password" {
  description = "Database password"
  type        = string
  sensitive   = true
}

resource "aws_db_instance" "main" {
  password = var.db_password
}
```

**2. Use AWS Secrets Manager**
```hcl
resource "aws_secretsmanager_secret" "db_password" {
  name = "${var.project}-db-password"
}

resource "aws_secretsmanager_secret_version" "db_password" {
  secret_id     = aws_secretsmanager_secret.db_password.id
  secret_string = var.db_password
}

data "aws_secretsmanager_secret_version" "db_password" {
  secret_id = aws_secretsmanager_secret.db_password.id
}

resource "aws_db_instance" "main" {
  password = jsondecode(data.aws_secretsmanager_secret_version.db_password.secret_string)["password"]
}
```

**3. Least Privilege IAM**
```hcl
resource "aws_iam_role" "ec2_role" {
  name = "${var.project}-ec2-role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Action = "sts:AssumeRole"
        Effect = "Allow"
        Principal = {
          Service = "ec2.amazonaws.com"
        }
      }
    ]
  })
}

resource "aws_iam_role_policy" "ec2_s3_access" {
  name = "${var.project}-ec2-s3-access"
  role = aws_iam_role.ec2_role.id

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect = "Allow"
        Action = [
          "s3:GetObject"
        ]
        Resource = "arn:aws:s3:::${var.project}-bucket/*"
      }
    ]
  })
}
```

### Testing and Validation

**1. Use terraform validate**
```bash
terraform validate
```

**2. Use terraform plan**
```bash
terraform plan -out=tfplan
terraform show tfplan
```

**3. Use terraform fmt**
```bash
terraform fmt -recursive
```

**4. Use external validation tools**
```bash
# Install tflint
curl -s https://raw.githubusercontent.com/terraform-linters/tflint/master/install_linux.sh | bash

# Run tflint
tflint

# Install checkov for security scanning
pip install checkov

# Run checkov
checkov -d .
```

### CI/CD Integration

**GitHub Actions Example:**
```yaml
name: Terraform

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  terraform:
    runs-on: ubuntu-latest
    
    steps:
    - name: Checkout
      uses: actions/checkout@v3
      
    - name: Setup Terraform
      uses: hashicorp/setup-terraform@v2
      with:
        terraform_version: 1.5.0
        
    - name: Configure AWS credentials
      uses: aws-actions/configure-aws-credentials@v2
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-region: us-west-2
        
    - name: Terraform Format
      run: terraform fmt -check
      
    - name: Terraform Init
      run: terraform init
      
    - name: Terraform Validate
      run: terraform validate
      
    - name: Terraform Plan
      run: terraform plan -no-color
      
    - name: Terraform Apply
      if: github.ref == 'refs/heads/main' && github.event_name == 'push'
      run: terraform apply -auto-approve
```

## Advanced Topics

### Workspaces

Workspaces allow you to manage multiple environments with the same configuration.

```bash
# List workspaces
terraform workspace list

# Create new workspace
terraform workspace new development
terraform workspace new production

# Switch workspace
terraform workspace select development

# Show current workspace
terraform workspace show

# Delete workspace
terraform workspace delete development
```

**Using workspaces in configuration:**
```hcl
locals {
  environment = terraform.workspace
  
  instance_type = terraform.workspace == "production" ? "t3.large" : "t3.micro"
  
  common_tags = {
    Environment = local.environment
    Workspace   = terraform.workspace
  }
}

resource "aws_instance" "web" {
  ami           = data.aws_ami.amazon_linux.id
  instance_type = local.instance_type
  
  tags = merge(local.common_tags, {
    Name = "web-${local.environment}"
  })
}
```

### Dynamic Blocks

Dynamic blocks allow you to generate repeated nested blocks.

```hcl
variable "ingress_rules" {
  type = list(object({
    from_port   = number
    to_port     = number
    protocol    = string
    cidr_blocks = list(string)
  }))
  
  default = [
    {
      from_port   = 80
      to_port     = 80
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
    },
    {
      from_port   = 443
      to_port     = 443
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
    }
  ]
}

resource "aws_security_group" "web" {
  name_prefix = "web-"
  vpc_id      = aws_vpc.main.id

  dynamic "ingress" {
    for_each = var.ingress_rules
    content {
      from_port   = ingress.value.from_port
      to_port     = ingress.value.to_port
      protocol    = ingress.value.protocol
      cidr_blocks = ingress.value.cidr_blocks
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

### Provisioners

Provisioners are used to execute scripts on local or remote machines.

```hcl
resource "aws_instance" "web" {
  ami           = data.aws_ami.amazon_linux.id
  instance_type = "t3.micro"
  key_name      = "my-key"
  
  # File provisioner
  provisioner "file" {
    source      = "app.conf"
    destination = "/tmp/app.conf"
    
    connection {
      type        = "ssh"
      user        = "ec2-user"
      private_key = file("my-key.pem")
      host        = self.public_ip
    }
  }
  
  # Remote-exec provisioner
  provisioner "remote-exec" {
    inline = [
      "sudo yum update -y",
      "sudo yum install -y httpd",
      "sudo systemctl start httpd",
      "sudo systemctl enable httpd",
      "sudo mv /tmp/app.conf /etc/httpd/conf.d/"
    ]
    
    connection {
      type        = "ssh"
      user        = "ec2-user"
      private_key = file("my-key.pem")
      host        = self.public_ip
    }
  }
  
  # Local-exec provisioner
  provisioner "local-exec" {
    command = "echo ${self.public_ip} > ip_address.txt"
  }
}
```

### Data Sources and External Data

```hcl
# External data source
data "external" "example" {
  program = ["python", "${path.module}/scripts/get_data.py"]
  
  query = {
    environment = var.environment
    region      = var.region
  }
}

resource "aws_instance" "web" {
  ami           = data.external.example.result.ami_id
  instance_type = data.external.example.result.instance_type
}
```

**scripts/get_data.py:**
```python
#!/usr/bin/env python3
import json
import sys

# Read input from Terraform
input_data = json.loads(sys.stdin.read())
environment = input_data.get('environment', 'dev')
region = input_data.get('region', 'us-west-2')

# Process data (example logic)
if environment == 'prod':
    ami_id = 'ami-prod-123'
    instance_type = 't3.large'
else:
    ami_id = 'ami-dev-456'
    instance_type = 't3.micro'

# Return result as JSON
result = {
    'ami_id': ami_id,
    'instance_type': instance_type
}

print(json.dumps(result))
```

### Import Existing Infrastructure

```bash
# Import existing resource
terraform import aws_instance.web i-1234567890abcdef0

# Import with module
terraform import module.web_server.aws_instance.web i-1234567890abcdef0

# Write configuration for imported resource
# (Check terraform show for current configuration)
resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1d0"
  instance_type = "t3.micro"
  # ... other required attributes
}
```

## Next Steps

### Learning Path

1. **Start with Basics**
   - Install Terraform
   - Create simple configurations
   - Practice with local state

2. **Intermediate Concepts**
   - Use modules
   - Implement remote state
   - Learn variable patterns

3. **Advanced Practices**
   - CI/CD integration
   - Multi-environment management
   - Security best practices

4. **Production Readiness**
   - State management strategies
   - Disaster recovery
   - Team collaboration

### Recommended Resources

**Official Resources:**
- [Terraform Documentation](https://www.terraform.io/docs/)
- [Terraform Registry](https://registry.terraform.io/)
- [HashiCorp Learn](https://learn.hashicorp.com/terraform)

**Community Resources:**
- [Terraform Best Practices](https://www.terraform-best-practices.com/)
- [Awesome Terraform](https://github.com/shuaibiyy/awesome-terraform)
- [Terraform AWS Modules](https://github.com/terraform-aws-modules)

**Practice Projects:**
1. Deploy a simple web application
2. Create a multi-tier architecture
3. Implement CI/CD pipeline with Terraform
4. Build disaster recovery setup
5. Create reusable modules library

### Certifications

- **HashiCorp Certified: Terraform Associate**
  - Validates foundational knowledge
  - Hands-on experience recommended
  - Covers basic to intermediate concepts

Remember: The best way to learn Terraform is through hands-on practice. Start with simple configurations and gradually build more complex infrastructure as you become comfortable with the concepts and tools!