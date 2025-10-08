# DevOps Advamced - IaC Fundamentals - Hands-On Laboratory

## Table of Contents
1. [Introduction to Terraform](#introduction)
2. [Lab Setup](#lab-setup)
3. [Lab 1: First Steps with Terraform](#lab-1-first-steps-with-terraform)
4. [Lab 2: Working with AWS EC2](#lab-2-working-with-aws-ec2)
5. [Lab 3: Managing S3 Buckets](#lab-3-managing-s3-buckets)
6. [Lab 4: VPC and Networking](#lab-4-vpc-and-networking)
7. [Lab 5: Variables and Outputs](#lab-5-variables-and-outputs)
8. [Lab 6: Terraform State Management](#lab-6-terraform-state-management)
9. [Lab 7: Modules and Reusability](#lab-7-modules-and-reusability)
10. [Best Practices and Next Steps](#best-practices-and-next-steps)

---

## Introduction to Terraform

### What is Terraform?

Terraform is an Infrastructure as Code (IaC) tool created by HashiCorp that allows you to define and provision infrastructure using a declarative configuration language called HashiCorp Configuration Language (HCL).

**Key Concepts:**
- **Declarative Language**: You describe the desired state, and Terraform figures out how to achieve it
- **Provider-Based**: Works with multiple cloud providers (AWS, Azure, GCP, etc.)
- **State Management**: Tracks the current state of your infrastructure
- **Idempotent**: Running the same configuration multiple times produces the same result

**Terraform Workflow:**
1. **Write**: Define infrastructure in `.tf` files
2. **Plan**: Preview changes before applying
3. **Apply**: Create/update infrastructure
4. **Destroy**: Remove infrastructure when needed

---

## Lab Setup

### Prerequisites Checklist

- [ ] AWS Account created
- [ ] AWS CLI installed
- [ ] AWS credentials configured (`aws configure`)
- [ ] Text editor installed (VS Code, Sublime, etc.)
- [ ] Basic understanding of cloud computing concepts

### Installation Steps

#### Installing Terraform on Linux/Mac

- [ ] Download Terraform from official website
  ```bash
  # For Linux
  wget https://releases.hashicorp.com/terraform/1.6.0/terraform_1.6.0_linux_amd64.zip
  unzip terraform_1.6.0_linux_amd64.zip
  sudo mv terraform /usr/local/bin/
  
  # For Mac (using Homebrew)
  brew tap hashicorp/tap
  brew install hashicorp/tap/terraform
  ```

- [ ] Verify installation
  ```bash
  terraform version
  ```

#### Installing Terraform on Windows

- [ ] Download the Windows executable from HashiCorp website
- [ ] Extract the zip file
- [ ] Add Terraform to your PATH environment variable
- [ ] Verify installation in PowerShell/CMD

  ```powershell
  terraform version
  ```

### AWS Credentials Setup

- [ ] Create IAM user with programmatic access
- [ ] Attach `AdministratorAccess` policy (for lab purposes only)
- [ ] Configure AWS CLI

  ```bash
  aws configure
  # Enter: Access Key ID, Secret Access Key, Region (e.g., us-east-1), Output format (json)
  ```

- [ ] Verify AWS credentials

  ```bash
  aws sts get-caller-identity
  ```

---

## Lab 1: First Steps with Terraform

### Theory: Terraform Basics

**Core Components:**
- **Providers**: Plugins that interact with cloud platforms (AWS, Azure, etc.)
- **Resources**: Infrastructure components you want to create (EC2, S3, etc.)
- **Data Sources**: Query existing infrastructure
- **Variables**: Parameterize your configurations
- **Outputs**: Display information after apply

**Terraform Commands:**
- `terraform init`: Initialize a working directory
- `terraform plan`: Preview changes
- `terraform apply`: Apply changes
- `terraform destroy`: Destroy managed infrastructure
- `terraform fmt`: Format code
- `terraform validate`: Validate configuration

### Practice: Your First Terraform Configuration

#### Step 1: Create Project Directory

- [ ] Create a new directory for your project

  ```bash
  mkdir terraform-labs
  cd terraform-labs
  mkdir lab1-basics
  cd lab1-basics
  ```

#### Step 2: Create Provider Configuration

- [ ] Create a file named `provider.tf`

  ```hcl
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
    region = "us-east-1"
  }
  ```

#### Step 3: Initialize Terraform

- [ ] Run terraform init

  ```bash
  terraform init
  ```

- [ ] Observe the output and verify provider installation
- [ ] Check that `.terraform` directory was created
- [ ] Review the `.terraform.lock.hcl` file

#### Step 4: Create a Simple Resource

- [ ] Create a file named `main.tf`

  ```hcl
  resource "aws_s3_bucket" "first_bucket" {
    bucket = "my-first-terraform-bucket-unique-name-12345"
    
    tags = {
      Name        = "My First Bucket"
      Environment = "Learning"
      ManagedBy   = "Terraform"
    }
  }
  ```

- [ ] Replace the bucket name with a globally unique name

#### Step 5: Plan and Apply

- [ ] Run terraform plan

  ```bash
  terraform plan
  ```

- [ ] Review the execution plan
- [ ] Understand what will be created (+ symbol)

- [ ] Apply the configuration

  ```bash
  terraform apply
  ```

- [ ] Type `yes` when prompted
- [ ] Wait for resource creation

#### Step 6: Verify Resource

- [ ] Check AWS Console for the new S3 bucket
- [ ] Or use AWS CLI

  ```bash
  aws s3 ls | grep my-first-terraform-bucket
  ```

#### Step 7: Understand State

- [ ] Examine the `terraform.tfstate` file
- [ ] Understand that this file tracks your infrastructure
- [ ] **Never manually edit this file**

#### Step 8: Clean Up

- [ ] Destroy the infrastructure

  ```bash
  terraform destroy
  ```

- [ ] Type `yes` when prompted
- [ ] Verify the bucket is deleted

---

## Lab 2: Working with AWS EC2

### Theory: EC2 Instances and Compute Resources

**Amazon EC2 (Elastic Compute Cloud):**
- Virtual servers in the cloud
- Various instance types for different workloads
- Requires: AMI (Amazon Machine Image), instance type, key pair, security group

**Terraform EC2 Resource Attributes:**
- `ami`: Amazon Machine Image ID
- `instance_type`: Size of the instance (t2.micro, t3.medium, etc.)
- `key_name`: SSH key pair name
- `vpc_security_group_ids`: Security groups for firewall rules
- `tags`: Metadata for organization

### Practice: Launch an EC2 Instance

#### Step 1: Create New Lab Directory

- [ ] Create directory for Lab 2

  ```bash
  cd ~/terraform-labs
  mkdir lab2-ec2
  cd lab2-ec2
  ```

#### Step 2: Create Key Pair

- [ ] Create SSH key pair in AWS

  ```bash
  aws ec2 create-key-pair --key-name terraform-lab-key --query 'KeyMaterial' --output text > terraform-lab-key.pem
  chmod 400 terraform-lab-key.pem
  ```

#### Step 3: Create Configuration Files

- [ ] Create `provider.tf`

  ```hcl
  terraform {
    required_providers {
      aws = {
        source  = "hashicorp/aws"
        version = "~> 5.0"
      }
    }
  }
  
  provider "aws" {
    region = "us-east-1"
  }
  ```

- [ ] Create `security-group.tf`

  ```hcl
  resource "aws_security_group" "web_sg" {
    name        = "terraform-web-sg"
    description = "Security group for web server"
  
    ingress {
      description = "SSH"
      from_port   = 22
      to_port     = 22
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
    }
  
    ingress {
      description = "HTTP"
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
      Name = "terraform-web-sg"
    }
  }
  ```

- [ ] Create `main.tf`

  ```hcl
  data "aws_ami" "ubuntu" {
    most_recent = true
    owners      = ["099720109477"] # Canonical
  
    filter {
      name   = "name"
      values = ["ubuntu/images/hvm-ssd/ubuntu-jammy-22.04-amd64-server-*"]
    }
  }
  
  resource "aws_instance" "web_server" {
    ami           = data.aws_ami.ubuntu.id
    instance_type = "t2.micro"
    key_name      = "terraform-lab-key"
    
    vpc_security_group_ids = [aws_security_group.web_sg.id]
  
    user_data = <<-EOF
                #!/bin/bash
                apt-get update
                apt-get install -y apache2
                systemctl start apache2
                systemctl enable apache2
                echo "<h1>Hello from Terraform!</h1>" > /var/www/html/index.html
                EOF
  
    tags = {
      Name = "terraform-web-server"
    }
  }
  ```

- [ ] Create `outputs.tf`

  ```hcl
  output "instance_id" {
    description = "ID of the EC2 instance"
    value       = aws_instance.web_server.id
  }
  
  output "instance_public_ip" {
    description = "Public IP address of the EC2 instance"
    value       = aws_instance.web_server.public_ip
  }
  
  output "instance_public_dns" {
    description = "Public DNS of the EC2 instance"
    value       = aws_instance.web_server.public_dns
  }
  ```

#### Step 4: Deploy Infrastructure

- [ ] Initialize Terraform

  ```bash
  terraform init
  ```

- [ ] Format the code

  ```bash
  terraform fmt
  ```

- [ ] Validate configuration

  ```bash
  terraform validate
  ```

- [ ] Plan deployment

  ```bash
  terraform plan
  ```

- [ ] Apply configuration

  ```bash
  terraform apply
  ```

#### Step 5: Verify Deployment

- [ ] Note the output values (IP address, DNS)
- [ ] Wait 2-3 minutes for user_data script to complete
- [ ] Open browser and navigate to the public IP
- [ ] Verify "Hello from Terraform!" message appears

- [ ] SSH into the instance

  ```bash
  ssh -i terraform-lab-key.pem ubuntu@<PUBLIC_IP>
  ```

#### Step 6: Make Changes

- [ ] Modify the user_data script in `main.tf`
- [ ] Run `terraform plan` to see changes
- [ ] Apply changes and observe instance replacement
- [ ] Understand immutable infrastructure concept

#### Step 7: Clean Up

- [ ] Destroy resources

  ```bash
  terraform destroy
  ```

---

## Lab 3: Managing S3 Buckets

### Theory: S3 and Object Storage

**Amazon S3 (Simple Storage Service):**
- Object storage service
- Buckets: Containers for objects
- Objects: Files stored in buckets
- Highly durable and available

**S3 Features in Terraform:**
- Bucket creation and configuration
- Versioning
- Lifecycle policies
- Bucket policies and ACLs
- Server-side encryption

### Practice: Create S3 Buckets with Advanced Features

#### Step 1: Setup

- [ ] Create lab directory

  ```bash
  cd ~/terraform-labs
  mkdir lab3-s3
  cd lab3-s3
  ```

#### Step 2: Create Configuration

- [ ] Create `provider.tf` (same as previous labs)

- [ ] Create `main.tf`

  ```hcl
  resource "aws_s3_bucket" "data_bucket" {
    bucket = "terraform-lab-data-bucket-${random_string.suffix.result}"
  
    tags = {
      Name        = "Data Bucket"
      Environment = "Learning"
    }
  }
  
  resource "random_string" "suffix" {
    length  = 8
    special = false
    upper   = false
  }
  
  resource "aws_s3_bucket_versioning" "data_bucket_versioning" {
    bucket = aws_s3_bucket.data_bucket.id
  
    versioning_configuration {
      status = "Enabled"
    }
  }
  
  resource "aws_s3_bucket_server_side_encryption_configuration" "data_bucket_encryption" {
    bucket = aws_s3_bucket.data_bucket.id
  
    rule {
      apply_server_side_encryption_by_default {
        sse_algorithm = "AES256"
      }
    }
  }
  
  resource "aws_s3_bucket_lifecycle_configuration" "data_bucket_lifecycle" {
    bucket = aws_s3_bucket.data_bucket.id
  
    rule {
      id     = "archive-old-objects"
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
  
  resource "aws_s3_bucket_public_access_block" "data_bucket_pab" {
    bucket = aws_s3_bucket.data_bucket.id
  
    block_public_acls       = true
    block_public_policy     = true
    ignore_public_acls      = true
    restrict_public_buckets = true
  }
  ```

- [ ] Create `outputs.tf`

  ```hcl
  output "bucket_name" {
    description = "Name of the S3 bucket"
    value       = aws_s3_bucket.data_bucket.id
  }
  
  output "bucket_arn" {
    description = "ARN of the S3 bucket"
    value       = aws_s3_bucket.data_bucket.arn
  }
  ```

#### Step 3: Deploy and Test

- [ ] Initialize and apply

  ```bash
  terraform init
  terraform apply
  ```

- [ ] Upload a test file

  ```bash
  echo "Hello Terraform" > test.txt
  aws s3 cp test.txt s3://$(terraform output -raw bucket_name)/
  ```

- [ ] Verify file in S3

  ```bash
  aws s3 ls s3://$(terraform output -raw bucket_name)/
  ```

- [ ] Update the file and upload again

  ```bash
  echo "Hello Terraform v2" > test.txt
  aws s3 cp test.txt s3://$(terraform output -raw bucket_name)/
  ```

- [ ] List versions

  ```bash
  aws s3api list-object-versions --bucket $(terraform output -raw bucket_name) --prefix test.txt
  ```

#### Step 4: Clean Up

- [ ] Empty the bucket first

  ```bash
  aws s3 rm s3://$(terraform output -raw bucket_name)/ --recursive
  ```

- [ ] Destroy infrastructure

  ```bash
  terraform destroy
  ```

---

## Lab 4: VPC and Networking

### Theory: AWS Networking

**VPC (Virtual Private Cloud):**
- Isolated network in AWS
- Subnets: Subdivisions of VPC (public/private)
- Internet Gateway: Allows internet access
- Route Tables: Control traffic routing
- NAT Gateway: Allows private subnet internet access

**CIDR Notation:**
- `10.0.0.0/16`: 65,536 IP addresses
- `10.0.1.0/24`: 256 IP addresses

### Practice: Build Complete Network Infrastructure

#### Step 1: Setup

- [ ] Create lab directory

  ```bash
  cd ~/terraform-labs
  mkdir lab4-vpc
  cd lab4-vpc
  ```

#### Step 2: Create VPC Configuration

- [ ] Create `provider.tf` (standard AWS provider)

- [ ] Create `vpc.tf`

  ```hcl
  resource "aws_vpc" "main" {
    cidr_block           = "10.0.0.0/16"
    enable_dns_hostnames = true
    enable_dns_support   = true
  
    tags = {
      Name = "terraform-vpc"
    }
  }
  
  resource "aws_internet_gateway" "main" {
    vpc_id = aws_vpc.main.id
  
    tags = {
      Name = "terraform-igw"
    }
  }
  ```

- [ ] Create `subnets.tf`

  ```hcl
  data "aws_availability_zones" "available" {
    state = "available"
  }
  
  resource "aws_subnet" "public_1" {
    vpc_id                  = aws_vpc.main.id
    cidr_block              = "10.0.1.0/24"
    availability_zone       = data.aws_availability_zones.available.names[0]
    map_public_ip_on_launch = true
  
    tags = {
      Name = "terraform-public-subnet-1"
    }
  }
  
  resource "aws_subnet" "public_2" {
    vpc_id                  = aws_vpc.main.id
    cidr_block              = "10.0.2.0/24"
    availability_zone       = data.aws_availability_zones.available.names[1]
    map_public_ip_on_launch = true
  
    tags = {
      Name = "terraform-public-subnet-2"
    }
  }
  
  resource "aws_subnet" "private_1" {
    vpc_id            = aws_vpc.main.id
    cidr_block        = "10.0.10.0/24"
    availability_zone = data.aws_availability_zones.available.names[0]
  
    tags = {
      Name = "terraform-private-subnet-1"
    }
  }
  
  resource "aws_subnet" "private_2" {
    vpc_id            = aws_vpc.main.id
    cidr_block        = "10.0.11.0/24"
    availability_zone = data.aws_availability_zones.available.names[1]
  
    tags = {
      Name = "terraform-private-subnet-2"
    }
  }
  ```

- [ ] Create `route-tables.tf`

  ```hcl
  resource "aws_route_table" "public" {
    vpc_id = aws_vpc.main.id
  
    route {
      cidr_block = "0.0.0.0/0"
      gateway_id = aws_internet_gateway.main.id
    }
  
    tags = {
      Name = "terraform-public-rt"
    }
  }
  
  resource "aws_route_table_association" "public_1" {
    subnet_id      = aws_subnet.public_1.id
    route_table_id = aws_route_table.public.id
  }
  
  resource "aws_route_table_association" "public_2" {
    subnet_id      = aws_subnet.public_2.id
    route_table_id = aws_route_table.public.id
  }
  
  resource "aws_route_table" "private" {
    vpc_id = aws_vpc.main.id
  
    tags = {
      Name = "terraform-private-rt"
    }
  }
  
  resource "aws_route_table_association" "private_1" {
    subnet_id      = aws_subnet.private_1.id
    route_table_id = aws_route_table.private.id
  }
  
  resource "aws_route_table_association" "private_2" {
    subnet_id      = aws_subnet.private_2.id
    route_table_id = aws_route_table.private.id
  }
  ```

- [ ] Create `outputs.tf`

  ```hcl
  output "vpc_id" {
    description = "ID of the VPC"
    value       = aws_vpc.main.id
  }
  
  output "public_subnet_ids" {
    description = "IDs of public subnets"
    value       = [aws_subnet.public_1.id, aws_subnet.public_2.id]
  }
  
  output "private_subnet_ids" {
    description = "IDs of private subnets"
    value       = [aws_subnet.private_1.id, aws_subnet.private_2.id]
  }
  ```

#### Step 3: Deploy and Verify

- [ ] Initialize and apply

  ```bash
  terraform init
  terraform apply
  ```

- [ ] Verify VPC in AWS Console
- [ ] Check subnets and route tables
- [ ] Review the VPC diagram in console

#### Step 4: Deploy Instance in VPC

- [ ] Create `ec2.tf`

  ```hcl
  data "aws_ami" "ubuntu" {
    most_recent = true
    owners      = ["099720109477"]
  
    filter {
      name   = "name"
      values = ["ubuntu/images/hvm-ssd/ubuntu-jammy-22.04-amd64-server-*"]
    }
  }
  
  resource "aws_security_group" "web" {
    name        = "terraform-web-sg"
    description = "Security group for web servers"
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
      cidr_blocks = ["0.0.0.0/0"]
    }
  
    egress {
      from_port   = 0
      to_port     = 0
      protocol    = "-1"
      cidr_blocks = ["0.0.0.0/0"]
    }
  
    tags = {
      Name = "terraform-web-sg"
    }
  }
  
  resource "aws_instance" "web" {
    ami                    = data.aws_ami.ubuntu.id
    instance_type          = "t2.micro"
    subnet_id              = aws_subnet.public_1.id
    vpc_security_group_ids = [aws_security_group.web.id]
  
    user_data = <<-EOF
                #!/bin/bash
                apt-get update
                apt-get install -y apache2
                echo "<h1>Instance in Custom VPC</h1>" > /var/www/html/index.html
                systemctl start apache2
                EOF
  
    tags = {
      Name = "terraform-vpc-web-server"
    }
  }
  ```

- [ ] Apply changes

  ```bash
  terraform apply
  ```

- [ ] Test web server accessibility

#### Step 5: Clean Up

- [ ] Destroy all resources

  ```bash
  terraform destroy
  ```

---

## Lab 5: Variables and Outputs

### Theory: Parameterizing Infrastructure

**Variables:**
- Make configurations reusable
- Types: string, number, bool, list, map, object
- Can have default values
- Can be validated
- Input from: CLI, files, environment variables

**Outputs:**
- Extract information from resources
- Can be used by other Terraform configurations
- Useful for displaying important information

### Practice: Create Parameterized Infrastructure

#### Step 1: Setup

- [ ] Create lab directory

  ```bash
  cd ~/terraform-labs
  mkdir lab5-variables
  cd lab5-variables
  ```

#### Step 2: Create Variable Definitions

- [ ] Create `variables.tf`

  ```hcl
  variable "aws_region" {
    description = "AWS region for resources"
    type        = string
    default     = "us-east-1"
  }
  
  variable "environment" {
    description = "Environment name"
    type        = string
    default     = "dev"
  
    validation {
      condition     = contains(["dev", "staging", "prod"], var.environment)
      error_message = "Environment must be dev, staging, or prod."
    }
  }
  
  variable "instance_type" {
    description = "EC2 instance type"
    type        = string
    default     = "t2.micro"
  }
  
  variable "instance_count" {
    description = "Number of instances to create"
    type        = number
    default     = 2
  
    validation {
      condition     = var.instance_count > 0 && var.instance_count <= 10
      error_message = "Instance count must be between 1 and 10."
    }
  }
  
  variable "enable_monitoring" {
    description = "Enable detailed monitoring"
    type        = bool
    default     = false
  }
  
  variable "allowed_ssh_ips" {
    description = "List of IPs allowed to SSH"
    type        = list(string)
    default     = ["0.0.0.0/0"]
  }
  
  variable "tags" {
    description = "Common tags for all resources"
    type        = map(string)
    default = {
      Project   = "Terraform Lab"
      ManagedBy = "Terraform"
    }
  }
  
  variable "server_config" {
    description = "Server configuration object"
    type = object({
      name          = string
      instance_type = string
      volume_size   = number
    })
    default = {
      name          = "web-server"
      instance_type = "t2.micro"
      volume_size   = 20
    }
  }
  ```

#### Step 3: Create Variable Values File

- [ ] Create `terraform.tfvars`

  ```hcl
  aws_region      = "us-east-1"
  environment     = "dev"
  instance_type   = "t2.micro"
  instance_count  = 2
  enable_monitoring = true
  allowed_ssh_ips = ["203.0.113.0/24"]
  
  tags = {
    Project     = "Variable Lab"
    ManagedBy   = "Terraform"
    Environment = "Development"
  }
  
  server_config = {
    name          = "app-server"
    instance_type = "t2.small"
    volume_size   = 30
  }
  ```

- [ ] Create `dev.tfvars` for development

  ```hcl
  environment    = "dev"
  instance_count = 2
  instance_type  = "t2.micro"
  ```

- [ ] Create `prod.tfvars` for production

  ```hcl
  environment       = "prod"
  instance_count    = 5
  instance_type     = "t3.medium"
  enable_monitoring = true
  ```

#### Step 4: Use Variables in Configuration

- [ ] Create `provider.tf`

  ```hcl
  terraform {
    required_providers {
      aws = {
        source  = "hashicorp/aws"
        version = "~> 5.0"
      }
    }
  }
  
  provider "aws" {
    region = var.aws_region
  }
  ```

- [ ] Create `main.tf`

  ```hcl
  data "aws_ami" "ubuntu" {
    most_recent = true
    owners      = ["099720109477"]
  
    filter {
      name   = "name"
      values = ["ubuntu/images/hvm-ssd/ubuntu-jammy-22.04-amd64-server-*"]
    }
  }
  
  resource "aws_security_group" "web" {
    name        = "${var.environment}-web-sg"
    description = "Security group for ${var.environment} web servers"
  
    ingress {
      from_port   = 22
      to_port     = 22
      protocol    = "tcp"
      cidr_blocks = var.allowed_ssh_ips
    }
  
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
  
    tags = merge(
      var.tags,
      {
        Name = "${var.environment}-web-sg"
      }
    )
  }
  
  resource "aws_instance" "web" {
    count                = var.instance_count
    ami                  = data.aws_ami.ubuntu.id
    instance_type        = var.instance_type
    monitoring           = var.enable_monitoring
    vpc_security_group_ids = [aws_security_group.web.id]
  
    root_block_device {
      volume_size = var.server_config.volume_size
      volume_type = "gp3"
    }
  
    user_data = <<-EOF
                #!/bin/bash
                apt-get update
                apt-get install -y apache2
                echo "<h1>${var.environment} Server ${count.index + 1}</h1>" > /var/www/html/index.html
                EOF
  
    tags = merge(
      var.tags,
      {
        Name = "${var.server_config.name}-${count.index + 1}"
        Environment = var.environment
      }
    )
  }
  ```

- [ ] Create `outputs.tf`

  ```hcl
  output "instance_ids" {
    description = "IDs of created instances"
    value       = aws_instance.web[*].id
  }
  
  output "instance_public_ips" {
    description = "Public IPs of instances"
    value       = aws_instance.web[*].public_ip
  }
  
  output "instance_public_dns" {
    description = "Public DNS names"
    value       = aws_instance.web[*].public_dns
  }
  
  output "security_group_id" {
    description = "ID of security group"
    value       = aws_security_group.web.id
  }
  
  output "environment_info" {
    description = "Environment information"
    value = {
      environment   = var.environment
      instance_type = var.instance_type
      instance_count = var.instance_count
      region        = var.aws_region
    }
  }
  ```

#### Step 5: Deploy with Different Configurations

- [ ] Deploy with default values

  ```bash
  terraform init
  terraform plan
  terraform apply
  ```

- [ ] Deploy with dev configuration

  ```bash
  terraform apply -var-file="dev.tfvars"
  ```

- [ ] Deploy with prod configuration (review plan only)

  ```bash
  terraform plan -var-file="prod.tfvars"
  ```

- [ ] Override variable from command line

  ```bash
  terraform apply -var="instance_count=3" -var="environment=dev"
  ```

#### Step 6: Work with Outputs

- [ ] View all outputs

  ```bash
  terraform output
  ```

- [ ] View specific output

  ```bash
  terraform output instance_public_ips
  ```

- [ ] Output in JSON format

  ```bash
  terraform output -json
  ```

#### Step 7: Clean Up

- [ ] Destroy resources

  ```bash
  terraform destroy
  ```

---

## Lab 6: Terraform State Management

### Theory: Understanding State

**Terraform State:**
- JSON file tracking managed resources
- Maps configuration to real-world resources
- Stores metadata and resource dependencies
- Can contain sensitive data

**State Management Best Practices:**
- Never manually edit state files
- Use remote state for teams
- Enable state locking
- Back up state files
- Use separate states for different environments

**Remote State Backends:**
- S3 with DynamoDB (AWS)
- Terraform Cloud
- Azure Blob Storage
- Google Cloud Storage

### Practice: Configure Remote State

#### Step 1: Setup

- [ ] Create lab directory

  ```bash
  cd ~/terraform-labs
  mkdir lab6-state
  cd lab6-state
  ```

#### Step 2: Create S3 Backend Resources

- [ ] Create `backend-setup` directory

  ```bash
  mkdir backend-setup
  cd backend-setup
  ```

- [ ] Create `main.tf` for backend resources

  ```hcl
  provider "aws" {
    region = "us-east-1"
  }
  
  resource "aws_s3_bucket" "terraform_state" {
    bucket = "terraform-state-${random_string.suffix.result}"
  
    tags = {
      Name        = "Terraform State Bucket"
      Environment = "Management"
    }
  }
  
  resource "random_string" "suffix" {
    length  = 8
    special = false
    upper   = false
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
        sse_algorithm = "AES256"
      }
    }
  }
  
  resource "aws_s3_bucket_public_access_block" "terraform_state" {
    bucket = aws_s3_bucket.terraform_state.id
  
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
      Name        = "Terraform State Lock Table"
      Environment = "Management"
    }
  }
  
  output "bucket_name" {
    description = "S3 bucket name for state"
    value       = aws_s3_bucket.terraform_state.id
  }
  
  output "dynamodb_table" {
    description = "DynamoDB table for state locking"
    value       = aws_dynamodb_table.terraform_locks.name
  }
  ```

- [ ] Deploy backend resources

  ```bash
  terraform init
  terraform apply
  ```

- [ ] Note the bucket name from output

  ```bash
  terraform output bucket_name
  ```

#### Step 3: Configure Remote Backend

- [ ] Go back to main lab directory

  ```bash
  cd ..
  ```

- [ ] Create `backend.tf` (replace BUCKET_NAME with your actual bucket name)

  ```hcl
  terraform {
    backend "s3" {
      bucket         = "BUCKET_NAME"  # Replace with your bucket name
      key            = "lab6/terraform.tfstate"
      region         = "us-east-1"
      dynamodb_table = "terraform-state-locks"
      encrypt        = true
    }
  }
  ```

- [ ] Create `provider.tf`

  ```hcl
  terraform {
    required_providers {
      aws = {
        source  = "hashicorp/aws"
        version = "~> 5.0"
      }
    }
  }
  
  provider "aws" {
    region = "us-east-1"
  }
  ```

- [ ] Create simple `main.tf`

  ```hcl
  resource "aws_s3_bucket" "example" {
    bucket = "example-bucket-${random_string.suffix.result}"
  
    tags = {
      Name = "Example Bucket"
    }
  }
  
  resource "random_string" "suffix" {
    length  = 8
    special = false
    upper   = false
  }
  ```

#### Step 4: Initialize with Remote Backend

- [ ] Initialize Terraform

  ```bash
  terraform init
  ```

- [ ] Observe backend configuration messages
- [ ] Verify no local `terraform.tfstate` file

- [ ] Apply configuration

  ```bash
  terraform apply
  ```

#### Step 5: Verify Remote State

- [ ] Check S3 bucket for state file

  ```bash
  aws s3 ls s3://YOUR-BUCKET-NAME/lab6/
  ```

- [ ] Download and view state (read-only)

  ```bash
  aws s3 cp s3://YOUR-BUCKET-NAME/lab6/terraform.tfstate - | jq .
  ```

#### Step 6: Test State Locking

- [ ] Open two terminal windows
- [ ] In first terminal, run:

  ```bash
  terraform apply -auto-approve
  ```

- [ ] Quickly in second terminal, try:

  ```bash
  terraform apply
  ```

- [ ] Observe lock error message
- [ ] Understand how locking prevents concurrent modifications

#### Step 7: State Commands

- [ ] List resources in state

  ```bash
  terraform state list
  ```

- [ ] Show specific resource

  ```bash
  terraform state show aws_s3_bucket.example
  ```

- [ ] Pull remote state to local

  ```bash
  terraform state pull > local-state-backup.json
  ```

#### Step 8: Migrate State (Optional)

- [ ] Create new backend configuration
- [ ] Run `terraform init -migrate-state`
- [ ] Confirm migration

#### Step 9: Clean Up

- [ ] Destroy main resources

  ```bash
  terraform destroy
  ```

- [ ] Go to backend-setup directory

  ```bash
  cd backend-setup
  ```

- [ ] Manually remove state files from S3 if needed

  ```bash
  aws s3 rm s3://YOUR-BUCKET-NAME/lab6/terraform.tfstate
  ```

- [ ] Destroy backend resources

  ```bash
  terraform destroy
  ```

---

## Lab 7: Modules and Reusability

### Theory: Terraform Modules

**What are Modules:**
- Containers for multiple resources used together
- Reusable components
- Can be sourced from: local paths, Terraform Registry, Git, HTTP URLs

**Module Structure:**
- Input variables: `variables.tf`
- Resources: `main.tf`
- Outputs: `outputs.tf`
- Optional: `README.md`, `versions.tf`

**Benefits:**
- DRY (Don't Repeat Yourself)
- Standardization
- Easier maintenance
- Collaboration

### Practice: Create and Use Modules

#### Step 1: Setup

- [ ] Create lab directory structure

  ```bash
  cd ~/terraform-labs
  mkdir -p lab7-modules/modules/{vpc,ec2}
  mkdir lab7-modules/environments/{dev,prod}
  cd lab7-modules
  ```

#### Step 2: Create VPC Module

- [ ] Create `modules/vpc/variables.tf`

  ```hcl
  variable "vpc_name" {
    description = "Name of the VPC"
    type        = string
  }
  
  variable "vpc_cidr" {
    description = "CIDR block for VPC"
    type        = string
    default     = "10.0.0.0/16"
  }
  
  variable "availability_zones" {
    description = "List of availability zones"
    type        = list(string)
  }
  
  variable "public_subnet_cidrs" {
    description = "CIDR blocks for public subnets"
    type        = list(string)
  }
  
  variable "private_subnet_cidrs" {
    description = "CIDR blocks for private subnets"
    type        = list(string)
  }
  
  variable "tags" {
    description = "Tags to apply to resources"
    type        = map(string)
    default     = {}
  }
  ```

- [ ] Create `modules/vpc/main.tf`

  ```hcl
  resource "aws_vpc" "main" {
    cidr_block           = var.vpc_cidr
    enable_dns_hostnames = true
    enable_dns_support   = true
  
    tags = merge(
      var.tags,
      {
        Name = var.vpc_name
      }
    )
  }
  
  resource "aws_internet_gateway" "main" {
    vpc_id = aws_vpc.main.id
  
    tags = merge(
      var.tags,
      {
        Name = "${var.vpc_name}-igw"
      }
    )
  }
  
  resource "aws_subnet" "public" {
    count                   = length(var.public_subnet_cidrs)
    vpc_id                  = aws_vpc.main.id
    cidr_block              = var.public_subnet_cidrs[count.index]
    availability_zone       = var.availability_zones[count.index]
    map_public_ip_on_launch = true
  
    tags = merge(
      var.tags,
      {
        Name = "${var.vpc_name}-public-${count.index + 1}"
        Type = "Public"
      }
    )
  }
  
  resource "aws_subnet" "private" {
    count             = length(var.private_subnet_cidrs)
    vpc_id            = aws_vpc.main.id
    cidr_block        = var.private_subnet_cidrs[count.index]
    availability_zone = var.availability_zones[count.index]
  
    tags = merge(
      var.tags,
      {
        Name = "${var.vpc_name}-private-${count.index + 1}"
        Type = "Private"
      }
    )
  }
  
  resource "aws_route_table" "public" {
    vpc_id = aws_vpc.main.id
  
    route {
      cidr_block = "0.0.0.0/0"
      gateway_id = aws_internet_gateway.main.id
    }
  
    tags = merge(
      var.tags,
      {
        Name = "${var.vpc_name}-public-rt"
      }
    )
  }
  
  resource "aws_route_table_association" "public" {
    count          = length(aws_subnet.public)
    subnet_id      = aws_subnet.public[count.index].id
    route_table_id = aws_route_table.public.id
  }
  ```

- [ ] Create `modules/vpc/outputs.tf`
  ```hcl
  output "vpc_id" {
    description = "ID of the VPC"
    value       = aws_vpc.main.id
  }
  
  output "vpc_cidr" {
    description = "CIDR block of VPC"
    value       = aws_vpc.main.cidr_block
  }
  
  output "public_subnet_ids" {
    description = "IDs of public subnets"
    value       = aws_subnet.public[*].id
  }
  
  output "private_subnet_ids" {
    description = "IDs of private subnets"
    value       = aws_subnet.private[*].id
  }
  
  output "internet_gateway_id" {
    description = "ID of Internet Gateway"
    value       = aws_internet_gateway.main.id
  }
  ```

#### Step 3: Create EC2 Module

- [ ] Create `modules/ec2/variables.tf`

  ```hcl
  variable "instance_name" {
    description = "Name of the instance"
    type        = string
  }
  
  variable "instance_type" {
    description = "EC2 instance type"
    type        = string
    default     = "t2.micro"
  }
  
  variable "subnet_id" {
    description = "Subnet ID for instance"
    type        = string
  }
  
  variable "vpc_id" {
    description = "VPC ID for security group"
    type        = string
  }
  
  variable "allowed_cidr_blocks" {
    description = "CIDR blocks allowed to access instance"
    type        = list(string)
    default     = ["0.0.0.0/0"]
  }
  
  variable "tags" {
    description = "Tags to apply to resources"
    type        = map(string)
    default     = {}
  }
  ```

- [ ] Create `modules/ec2/main.tf`

  ```hcl
  data "aws_ami" "ubuntu" {
    most_recent = true
    owners      = ["099720109477"]
  
    filter {
      name   = "name"
      values = ["ubuntu/images/hvm-ssd/ubuntu-jammy-22.04-amd64-server-*"]
    }
  }
  
  resource "aws_security_group" "instance" {
    name        = "${var.instance_name}-sg"
    description = "Security group for ${var.instance_name}"
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
  
    tags = merge(
      var.tags,
      {
        Name = "${var.instance_name}-sg"
      }
    )
  }
  
  resource "aws_instance" "main" {
    ami                    = data.aws_ami.ubuntu.id
    instance_type          = var.instance_type
    subnet_id              = var.subnet_id
    vpc_security_group_ids = [aws_security_group.instance.id]
  
    user_data = <<-EOF
                #!/bin/bash
                apt-get update
                apt-get install -y apache2
                echo "<h1>${var.instance_name}</h1>" > /var/www/html/index.html
                systemctl start apache2
                EOF
  
    tags = merge(
      var.tags,
      {
        Name = var.instance_name
      }
    )
  }
  ```

- [ ] Create `modules/ec2/outputs.tf`

  ```hcl
  output "instance_id" {
    description = "ID of the instance"
    value       = aws_instance.main.id
  }
  
  output "instance_public_ip" {
    description = "Public IP of instance"
    value       = aws_instance.main.public_ip
  }
  
  output "instance_public_dns" {
    description = "Public DNS of instance"
    value       = aws_instance.main.public_dns
  }
  
  output "security_group_id" {
    description = "ID of security group"
    value       = aws_security_group.instance.id
  }
  ```

#### Step 4: Use Modules in Dev Environment

- [ ] Create `environments/dev/provider.tf`

  ```hcl
  terraform {
    required_providers {
      aws = {
        source  = "hashicorp/aws"
        version = "~> 5.0"
      }
    }
  }
  
  provider "aws" {
    region = "us-east-1"
  }
  ```

- [ ] Create `environments/dev/main.tf`

  ```hcl
  data "aws_availability_zones" "available" {
    state = "available"
  }
  
  module "vpc" {
    source = "../../modules/vpc"
  
    vpc_name             = "dev-vpc"
    vpc_cidr             = "10.0.0.0/16"
    availability_zones   = slice(data.aws_availability_zones.available.names, 0, 2)
    public_subnet_cidrs  = ["10.0.1.0/24", "10.0.2.0/24"]
    private_subnet_cidrs = ["10.0.10.0/24", "10.0.11.0/24"]
  
    tags = {
      Environment = "Development"
      ManagedBy   = "Terraform"
      Project     = "Module Lab"
    }
  }
  
  module "web_server_1" {
    source = "../../modules/ec2"
  
    instance_name  = "dev-web-server-1"
    instance_type  = "t2.micro"
    subnet_id      = module.vpc.public_subnet_ids[0]
    vpc_id         = module.vpc.vpc_id
  
    tags = {
      Environment = "Development"
      ManagedBy   = "Terraform"
    }
  }
  
  module "web_server_2" {
    source = "../../modules/ec2"
  
    instance_name  = "dev-web-server-2"
    instance_type  = "t2.micro"
    subnet_id      = module.vpc.public_subnet_ids[1]
    vpc_id         = module.vpc.vpc_id
  
    tags = {
      Environment = "Development"
      ManagedBy   = "Terraform"
    }
  }
  ```

- [ ] Create `environments/dev/outputs.tf`

  ```hcl
  output "vpc_id" {
    description = "VPC ID"
    value       = module.vpc.vpc_id
  }
  
  output "web_server_1_ip" {
    description = "Web Server 1 Public IP"
    value       = module.web_server_1.instance_public_ip
  }
  
  output "web_server_2_ip" {
    description = "Web Server 2 Public IP"
    value       = module.web_server_2.instance_public_ip
  }
  ```

#### Step 5: Deploy Dev Environment

- [ ] Navigate to dev environment

  ```bash
  cd environments/dev
  ```

- [ ] Initialize and apply

  ```bash
  terraform init
  terraform plan
  terraform apply
  ```

- [ ] Verify outputs

  ```bash
  terraform output
  ```

- [ ] Test web servers

  ```bash
  curl http://$(terraform output -raw web_server_1_ip)
  curl http://$(terraform output -raw web_server_2_ip)
  ```

#### Step 6: Create Prod Environment

- [ ] Create `environments/prod/provider.tf` (same as dev)

- [ ] Create `environments/prod/main.tf`

  ```hcl
  data "aws_availability_zones" "available" {
    state = "available"
  }
  
  module "vpc" {
    source = "../../modules/vpc"
  
    vpc_name             = "prod-vpc"
    vpc_cidr             = "10.1.0.0/16"
    availability_zones   = slice(data.aws_availability_zones.available.names, 0, 3)
    public_subnet_cidrs  = ["10.1.1.0/24", "10.1.2.0/24", "10.1.3.0/24"]
    private_subnet_cidrs = ["10.1.10.0/24", "10.1.11.0/24", "10.1.12.0/24"]
  
    tags = {
      Environment = "Production"
      ManagedBy   = "Terraform"
      Project     = "Module Lab"
    }
  }
  
  module "web_server" {
    source = "../../modules/ec2"
  
    instance_name  = "prod-web-server"
    instance_type  = "t3.small"  # Larger instance for prod
    subnet_id      = module.vpc.public_subnet_ids[0]
    vpc_id         = module.vpc.vpc_id
  
    allowed_cidr_blocks = ["10.1.0.0/16"]  # More restrictive for prod
  
    tags = {
      Environment = "Production"
      ManagedBy   = "Terraform"
      Critical    = "Yes"
    }
  }
  ```

- [ ] Note the differences: larger instance type, more AZs, stricter security

#### Step 7: Understanding Module Benefits

- [ ] Compare dev and prod configurations
- [ ] Note how modules enable:
  - Consistent infrastructure patterns
  - Easy replication across environments
  - Centralized updates (change module once, affects all uses)
  - Clear separation of concerns

#### Step 8: Clean Up

- [ ] Destroy dev environment

  ```bash
  cd ~/terraform-labs/lab7-modules/environments/dev
  terraform destroy
  ```

- [ ] Optionally plan prod (don't apply)

  ```bash
  cd ~/terraform-labs/lab7-modules/environments/prod
  terraform plan
  ```

---

## Best Practices and Next Steps

### Terraform Best Practices Checklist

#### Code Organization
- [ ] Use consistent file naming (`main.tf`, `variables.tf`, `outputs.tf`)
- [ ] Separate provider configuration
- [ ] Use modules for reusable components
- [ ] Keep environments in separate directories
- [ ] Document your code with comments

#### State Management
- [ ] Always use remote state for team projects
- [ ] Enable state locking
- [ ] Enable state versioning/backup
- [ ] Never commit state files to version control
- [ ] Use separate states for different environments

#### Security
- [ ] Never commit credentials to version control
- [ ] Use AWS IAM roles when possible
- [ ] Implement least privilege access
- [ ] Encrypt state files at rest
- [ ] Use secret management tools (AWS Secrets Manager, HashiCorp Vault)
- [ ] Review security group rules regularly

#### Development Workflow
- [ ] Always run `terraform plan` before `apply`
- [ ] Use `terraform fmt` to format code
- [ ] Use `terraform validate` to check syntax
- [ ] Implement code review process
- [ ] Use version control (Git)
- [ ] Tag releases and use semantic versioning

#### Resource Management
- [ ] Use tags consistently across all resources
- [ ] Implement naming conventions
- [ ] Use `depends_on` only when necessary
- [ ] Leverage data sources instead of hardcoding
- [ ] Use lifecycle rules appropriately

#### Variables and Outputs
- [ ] Provide descriptions for all variables
- [ ] Set appropriate default values
- [ ] Use variable validation when possible
- [ ] Output useful information
- [ ] Mark sensitive outputs appropriately

### Common Terraform Commands Reference

```bash
# Initialize working directory
terraform init

# Validate configuration
terraform validate

# Format code
terraform fmt

# Plan changes
terraform plan
terraform plan -out=tfplan

# Apply changes
terraform apply
terraform apply tfplan
terraform apply -auto-approve

# Destroy infrastructure
terraform destroy
terraform destroy -auto-approve

# Show current state
terraform show

# List resources in state
terraform state list

# Show specific resource
terraform state show <resource>

# Import existing resource
terraform import <resource> <id>

# Refresh state
terraform refresh

# Output values
terraform output
terraform output <output_name>
terraform output -json

# Workspace management
terraform workspace list
terraform workspace new <name>
terraform workspace select <name>

# Graph
terraform graph | dot -Tsvg > graph.svg
```

### Next Steps in Your Terraform Journey

#### Advanced Topics to Explore
- [ ] Terraform Workspaces for environment management
- [ ] Dynamic blocks and complex expressions
- [ ] Custom providers
- [ ] Terraform Cloud and Enterprise features
- [ ] CI/CD integration with Terraform
- [ ] Testing Terraform code (terratest)
- [ ] Sentinel policies for governance

#### Additional Learning Resources
- [ ] [Terraform Documentation](https://www.terraform.io/docs)
- [ ] [Terraform Registry](https://registry.terraform.io/)
- [ ] [HashiCorp Learn](https://learn.hashicorp.com/terraform)

### Congratulations!

You've completed the Terraform Fundamentals laboratory. You now have hands-on experience with:
- Installing and configuring Terraform
- Managing AWS resources with Infrastructure as Code
- Working with variables, outputs, and modules
- Understanding state management
- Following best practices

Keep practicing and building more complex infrastructures. The key to mastering Terraform is consistent practice and real-world application.

Kirk Patrick

---

**Lab Version:** 1.8  
**Last Updated:** 2025  
**Terraform Version:** 1.6+
