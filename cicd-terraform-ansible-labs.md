# CI/CD Pipeline - Hands-On Lab
## Terraform + Ansible + GitHub Actions

## Table of Contents
1. [Introduction to CI/CD](#introduction-to-cicd)
2. [Lab Setup](#lab-setup)
3. [Lab 1: Terraform Basics for CI/CD](#lab-1-terraform-basics-for-cicd)
4. [Lab 2: Building Infrastructure with Terraform](#lab-2-building-infrastructure-with-terraform)
5. [Lab 3: Server Configuration with Ansible](#lab-3-server-configuration-with-ansible)
6. [Lab 4: Introduction to GitHub Actions](#lab-4-introduction-to-github-actions)
7. [Lab 5: Complete CI/CD Pipeline](#lab-5-complete-cicd-pipeline)
8. [Lab 6: Advanced Pipeline Features](#lab-6-advanced-pipeline-features)
9. [Lab 7: Monitoring and Troubleshooting](#lab-7-monitoring-and-troubleshooting)
10. [Best Practices and Next Steps](#conclusion)

---

## Introduction to CI/CD

### What is CI/CD?

CI/CD stands for Continuous Integration and Continuous Deployment/Delivery. It's a methodology that automates the building, testing, and deployment of applications and infrastructure.

**Key Concepts:**
- **Continuous Integration**: Automatically testing code changes
- **Continuous Deployment**: Automatically deploying changes to production
- **Infrastructure as Code**: Managing infrastructure through code
- **Configuration Management**: Automating server configuration

### The Three Pillars of Our Pipeline

#### Terraform
**Purpose:** Infrastructure provisioning
- Declarative infrastructure definition
- Cloud-agnostic (works with AWS, Azure, GCP)
- State management for tracking resources
- Idempotent operations

#### Ansible
**Purpose:** Configuration management
- Agentless architecture (SSH-based)
- YAML-based playbooks
- Idempotent task execution
- Extensive module library

#### GitHub Actions
**Purpose:** CI/CD orchestration
- Event-driven workflows
- Native GitHub integration
- Matrix builds and parallel jobs
- Extensive marketplace

### Pipeline Flow

```
Code Change → GitHub → GitHub Actions → Terraform (Provision) → Ansible (Configure) → Deployment
```

**Workflow Steps:**
1. Developer pushes code to GitHub
2. GitHub Actions workflow triggers
3. Terraform provisions infrastructure
4. Ansible configures servers
5. Application deploys
6. Tests run automatically
7. Notifications sent

---

## Lab Setup

### Prerequisites Checklist

- [ ] AWS Account with valid credentials
- [ ] GitHub account created
- [ ] Local machine with terminal access
- [ ] Text editor installed (VS Code recommended)
- [ ] Basic understanding of Linux commands
- [ ] Basic Git knowledge

### Required Tools Installation

#### Installing Terraform

**Linux:**
- [ ] Download and install Terraform
  ```bash
  wget https://releases.hashicorp.com/terraform/1.6.0/terraform_1.6.0_linux_amd64.zip
  unzip terraform_1.6.0_linux_amd64.zip
  sudo mv terraform /usr/local/bin/
  terraform version
  ```

**macOS:**
- [ ] Install via Homebrew
  ```bash
  brew tap hashicorp/tap
  brew install hashicorp/tap/terraform
  terraform version
  ```

**Windows:**
- [ ] Download from HashiCorp website
- [ ] Extract to a folder (e.g., `C:\terraform`)
- [ ] Add to PATH environment variable
- [ ] Verify in PowerShell
  ```powershell
  terraform version
  ```

#### Installing Ansible

**Linux/macOS:**
- [ ] Install via pip
  ```bash
  # Install pip if needed
  curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py
  python3 get-pip.py
  
  # Install Ansible
  pip3 install ansible
  ansible --version
  ```

**Windows (WSL2 recommended):**
- [ ] Enable WSL2
- [ ] Install Ubuntu from Microsoft Store
- [ ] Follow Linux instructions above

#### Installing AWS CLI

- [ ] Download and install AWS CLI
  ```bash
  # Linux
  curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
  unzip awscliv2.zip
  sudo ./aws/install
  
  # macOS
  brew install awscli
  
  # Verify
  aws --version
  ```

#### Installing Git

- [ ] Install Git
  ```bash
  # Linux (Debian/Ubuntu)
  sudo apt-get update
  sudo apt-get install git
  
  # macOS
  brew install git
  
  # Windows
  # Download from git-scm.com
  
  # Verify
  git --version
  ```

### AWS Configuration

- [ ] Create IAM user with programmatic access
  ```bash
  # Login to AWS Console
  # Navigate to IAM → Users → Add User
  # Enable "Programmatic access"
  # Attach policy: AdministratorAccess (for lab only)
  # Save Access Key ID and Secret Access Key
  ```

- [ ] Configure AWS credentials
  ```bash
  aws configure
  # AWS Access Key ID: [your-access-key]
  # AWS Secret Access Key: [your-secret-key]
  # Default region: us-east-1
  # Default output format: json
  ```

- [ ] Verify configuration
  ```bash
  aws sts get-caller-identity
  ```

### GitHub Configuration

- [ ] Create GitHub account at github.com
- [ ] Install GitHub CLI (optional but recommended)
  ```bash
  # Linux
  curl -fsSL https://cli.github.com/packages/githubcli-archive-keyring.gpg | sudo dd of=/usr/share/keyrings/githubcli-archive-keyring.gpg
  echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" | sudo tee /etc/apt/sources.list.d/github-cli.list > /dev/null
  sudo apt update
  sudo apt install gh
  
  # macOS
  brew install gh
  
  # Authenticate
  gh auth login
  ```

### SSH Key Setup

- [ ] Generate SSH key pair for AWS EC2
  ```bash
  ssh-keygen -t rsa -b 4096 -f ~/.ssh/cicd-lab-key -N ""
  chmod 400 ~/.ssh/cicd-lab-key
  ```

- [ ] Import key to AWS
  ```bash
  aws ec2 import-key-pair \
    --key-name cicd-lab-key \
    --public-key-material fileb://~/.ssh/cicd-lab-key.pub \
    --region us-east-1
  ```

### Project Directory Setup

- [ ] Create project structure
  ```bash
  mkdir -p ~/cicd-pipeline-lab
  cd ~/cicd-pipeline-lab
  
  mkdir -p {terraform,ansible/{inventory,playbooks,templates},scripts}
  touch terraform/{main.tf,variables.tf,outputs.tf,provider.tf}
  touch ansible/{ansible.cfg,playbooks/configure.yml}
  
  tree
  ```

---

## Lab 1: Terraform Basics for CI/CD

### Theory: Infrastructure as Code for CI/CD

**Why Terraform for CI/CD?**
- Reproducible infrastructure
- Version-controlled infrastructure
- Easy rollback capabilities
- Environment consistency
- Automated provisioning

**Terraform in CI/CD Context:**
- CI/CD pipelines provision infrastructure on-demand
- Infrastructure changes are code-reviewed
- State management enables tracking
- Outputs feed into configuration management

**Best Practices for CI/CD:**
- Use remote state storage (S3 + DynamoDB)
- Lock state files during operations
- Separate environments with workspaces or directories
- Tag all resources consistently
- Use variables for flexibility

### Practice: Create Basic Terraform Configuration

#### Step 1: Initialize Lab Directory

- [ ] Create Lab 1 directory
  ```bash
  cd ~/cicd-pipeline-lab
  mkdir lab1-terraform-basics
  cd lab1-terraform-basics
  ```

#### Step 2: Create Provider Configuration

- [ ] Create `provider.tf`
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
    region = var.aws_region
    
    default_tags {
      tags = {
        ManagedBy   = "Terraform"
        Environment = var.environment
        Project     = "CICD-Pipeline-Lab"
      }
    }
  }
  ```

#### Step 3: Define Variables

- [ ] Create `variables.tf`
  ```hcl
  variable "aws_region" {
    description = "AWS region for resources"
    type        = string
    default     = "us-east-1"
  }
  
  variable "environment" {
    description = "Environment name (dev, staging, prod)"
    type        = string
    default     = "dev"
    
    validation {
      condition     = contains(["dev", "staging", "prod"], var.environment)
      error_message = "Environment must be dev, staging, or prod."
    }
  }
  
  variable "project_name" {
    description = "Project name for resource naming"
    type        = string
    default     = "cicd-lab"
  }
  ```

- [ ] Create `terraform.tfvars`
  ```hcl
  aws_region   = "us-east-1"
  environment  = "dev"
  project_name = "cicd-lab"
  ```

#### Step 4: Create Simple Resource

- [ ] Create `main.tf`
  ```hcl
  # Random suffix for unique naming
  resource "random_string" "suffix" {
    length  = 8
    special = false
    upper   = false
  }
  
  # S3 bucket for testing
  resource "aws_s3_bucket" "test" {
    bucket = "${var.project_name}-${var.environment}-test-${random_string.suffix.result}"
    
    tags = {
      Name = "${var.project_name}-test-bucket"
    }
  }
  
  # Enable versioning
  resource "aws_s3_bucket_versioning" "test" {
    bucket = aws_s3_bucket.test.id
    
    versioning_configuration {
      status = "Enabled"
    }
  }
  ```

#### Step 5: Define Outputs

- [ ] Create `outputs.tf`
  ```hcl
  output "bucket_name" {
    description = "Name of the created S3 bucket"
    value       = aws_s3_bucket.test.id
  }
  
  output "bucket_arn" {
    description = "ARN of the S3 bucket"
    value       = aws_s3_bucket.test.arn
  }
  
  output "bucket_region" {
    description = "Region where bucket is created"
    value       = aws_s3_bucket.test.region
  }
  ```

#### Step 6: Initialize and Validate

- [ ] Initialize Terraform
  ```bash
  terraform init
  ```
  
- [ ] Review initialization output
- [ ] Check `.terraform` directory creation
- [ ] Review `.terraform.lock.hcl`

- [ ] Format code
  ```bash
  terraform fmt
  ```

- [ ] Validate configuration
  ```bash
  terraform validate
  ```

#### Step 7: Plan and Apply

- [ ] Generate execution plan
  ```bash
  terraform plan
  ```

- [ ] Review the plan output
- [ ] Identify resources to be created (+ symbol)
- [ ] Check resource attributes

- [ ] Save plan to file
  ```bash
  terraform plan -out=tfplan
  ```

- [ ] Apply the plan
  ```bash
  terraform apply tfplan
  ```

- [ ] Review outputs
  ```bash
  terraform output
  terraform output -json
  ```

#### Step 8: Verify Resources

- [ ] Check AWS Console for S3 bucket
- [ ] Use AWS CLI
  ```bash
  aws s3 ls | grep cicd-lab
  aws s3api get-bucket-versioning --bucket $(terraform output -raw bucket_name)
  ```

#### Step 9: Understand State

- [ ] Examine state file
  ```bash
  cat terraform.tfstate | jq '.'
  ```

- [ ] List resources in state
  ```bash
  terraform state list
  ```

- [ ] Show specific resource
  ```bash
  terraform state show aws_s3_bucket.test
  ```

#### Step 10: Make Changes

- [ ] Modify `main.tf` to add encryption
  ```hcl
  resource "aws_s3_bucket_server_side_encryption_configuration" "test" {
    bucket = aws_s3_bucket.test.id
    
    rule {
      apply_server_side_encryption_by_default {
        sse_algorithm = "AES256"
      }
    }
  }
  ```

- [ ] Plan and apply changes
  ```bash
  terraform plan
  terraform apply
  ```

- [ ] Observe the update process

#### Step 11: Clean Up

- [ ] Destroy infrastructure
  ```bash
  terraform destroy
  ```

- [ ] Type `yes` when prompted
- [ ] Verify resources are deleted
  ```bash
  aws s3 ls | grep cicd-lab
  ```

---

## Lab 2: Building Infrastructure with Terraform

### Theory: Complete Infrastructure Stack

**Infrastructure Components:**
- **VPC**: Network isolation
- **Subnets**: Network segmentation (public/private)
- **Security Groups**: Firewall rules
- **EC2 Instances**: Compute resources
- **Internet Gateway**: External connectivity

**CI/CD Infrastructure Requirements:**
- Reproducible across environments
- Secure by default
- Scalable architecture
- Properly tagged for management
- Outputs for automation

### Practice: Build Complete Infrastructure

#### Step 1: Create Lab Directory

- [ ] Setup Lab 2
  ```bash
  cd ~/cicd-pipeline-lab
  mkdir lab2-infrastructure
  cd lab2-infrastructure
  ```

#### Step 2: Create VPC Configuration

- [ ] Create `provider.tf`
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
    region = var.aws_region
  }
  ```

- [ ] Create `variables.tf`
  ```hcl
  variable "aws_region" {
    description = "AWS region"
    type        = string
    default     = "us-east-1"
  }
  
  variable "environment" {
    description = "Environment name"
    type        = string
    default     = "dev"
  }
  
  variable "project_name" {
    description = "Project name"
    type        = string
    default     = "cicd-pipeline"
  }
  
  variable "vpc_cidr" {
    description = "CIDR block for VPC"
    type        = string
    default     = "10.0.0.0/16"
  }
  
  variable "public_subnet_cidrs" {
    description = "CIDR blocks for public subnets"
    type        = list(string)
    default     = ["10.0.1.0/24", "10.0.2.0/24"]
  }
  
  variable "instance_type" {
    description = "EC2 instance type"
    type        = string
    default     = "t2.micro"
  }
  
  variable "instance_count" {
    description = "Number of instances"
    type        = number
    default     = 2
  }
  
  variable "ssh_key_name" {
    description = "SSH key pair name"
    type        = string
    default     = "cicd-lab-key"
  }
  ```

- [ ] Create `vpc.tf`
  ```hcl
  # VPC
  resource "aws_vpc" "main" {
    cidr_block           = var.vpc_cidr
    enable_dns_hostnames = true
    enable_dns_support   = true
    
    tags = {
      Name = "${var.project_name}-${var.environment}-vpc"
    }
  }
  
  # Internet Gateway
  resource "aws_internet_gateway" "main" {
    vpc_id = aws_vpc.main.id
    
    tags = {
      Name = "${var.project_name}-${var.environment}-igw"
    }
  }
  
  # Data source for availability zones
  data "aws_availability_zones" "available" {
    state = "available"
  }
  
  # Public Subnets
  resource "aws_subnet" "public" {
    count                   = length(var.public_subnet_cidrs)
    vpc_id                  = aws_vpc.main.id
    cidr_block              = var.public_subnet_cidrs[count.index]
    availability_zone       = data.aws_availability_zones.available.names[count.index]
    map_public_ip_on_launch = true
    
    tags = {
      Name = "${var.project_name}-${var.environment}-public-subnet-${count.index + 1}"
      Type = "Public"
    }
  }
  
  # Route Table
  resource "aws_route_table" "public" {
    vpc_id = aws_vpc.main.id
    
    route {
      cidr_block = "0.0.0.0/0"
      gateway_id = aws_internet_gateway.main.id
    }
    
    tags = {
      Name = "${var.project_name}-${var.environment}-public-rt"
    }
  }
  
  # Route Table Associations
  resource "aws_route_table_association" "public" {
    count          = length(aws_subnet.public)
    subnet_id      = aws_subnet.public[count.index].id
    route_table_id = aws_route_table.public.id
  }
  ```

#### Step 3: Create Security Groups

- [ ] Create `security-groups.tf`
  ```hcl
  # Security Group for Web Servers
  resource "aws_security_group" "web" {
    name        = "${var.project_name}-${var.environment}-web-sg"
    description = "Security group for web servers"
    vpc_id      = aws_vpc.main.id
    
    # SSH access
    ingress {
      description = "SSH from anywhere"
      from_port   = 22
      to_port     = 22
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
    }
    
    # HTTP access
    ingress {
      description = "HTTP from anywhere"
      from_port   = 80
      to_port     = 80
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
    }
    
    # HTTPS access
    ingress {
      description = "HTTPS from anywhere"
      from_port   = 443
      to_port     = 443
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
    }
    
    # Outbound traffic
    egress {
      description = "Allow all outbound"
      from_port   = 0
      to_port     = 0
      protocol    = "-1"
      cidr_blocks = ["0.0.0.0/0"]
    }
    
    tags = {
      Name = "${var.project_name}-${var.environment}-web-sg"
    }
  }
  ```

#### Step 4: Create EC2 Instances

- [ ] Create `ec2.tf`
  ```hcl
  # Data source for latest Ubuntu AMI
  data "aws_ami" "ubuntu" {
    most_recent = true
    owners      = ["099720109477"] # Canonical
    
    filter {
      name   = "name"
      values = ["ubuntu/images/hvm-ssd/ubuntu-jammy-22.04-amd64-server-*"]
    }
    
    filter {
      name   = "virtualization-type"
      values = ["hvm"]
    }
  }
  
  # EC2 Instances
  resource "aws_instance" "web" {
    count                  = var.instance_count
    ami                    = data.aws_ami.ubuntu.id
    instance_type          = var.instance_type
    subnet_id              = aws_subnet.public[count.index % length(aws_subnet.public)].id
    vpc_security_group_ids = [aws_security_group.web.id]
    key_name               = var.ssh_key_name
    
    user_data = <<-EOF
                #!/bin/bash
                apt-get update
                apt-get install -y nginx python3 python3-pip
                systemctl start nginx
                systemctl enable nginx
                echo "<h1>Server ${count.index + 1} - ${var.environment}</h1>" > /var/www/html/index.html
                echo "<p>Hostname: $(hostname)</p>" >> /var/www/html/index.html
                EOF
    
    tags = {
      Name = "${var.project_name}-${var.environment}-web-${count.index + 1}"
      Role = "webserver"
    }
  }
  ```

#### Step 5: Create Outputs

- [ ] Create `outputs.tf`
  ```hcl
  output "vpc_id" {
    description = "VPC ID"
    value       = aws_vpc.main.id
  }
  
  output "vpc_cidr" {
    description = "VPC CIDR block"
    value       = aws_vpc.main.cidr_block
  }
  
  output "public_subnet_ids" {
    description = "Public subnet IDs"
    value       = aws_subnet.public[*].id
  }
  
  output "security_group_id" {
    description = "Security group ID"
    value       = aws_security_group.web.id
  }
  
  output "instance_ids" {
    description = "EC2 instance IDs"
    value       = aws_instance.web[*].id
  }
  
  output "instance_public_ips" {
    description = "Public IPs of instances"
    value       = aws_instance.web[*].public_ip
  }
  
  output "instance_public_dns" {
    description = "Public DNS names of instances"
    value       = aws_instance.web[*].public_dns
  }
  
  output "web_urls" {
    description = "URLs to access web servers"
    value       = [for ip in aws_instance.web[*].public_ip : "http://${ip}"]
  }
  ```

#### Step 6: Deploy Infrastructure

- [ ] Initialize Terraform
  ```bash
  terraform init
  ```

- [ ] Validate configuration
  ```bash
  terraform validate
  ```

- [ ] Format code
  ```bash
  terraform fmt -recursive
  ```

- [ ] Generate plan
  ```bash
  terraform plan
  ```

- [ ] Review the plan thoroughly
- [ ] Count resources to be created

- [ ] Apply configuration
  ```bash
  terraform apply
  ```

- [ ] Type `yes` to confirm

#### Step 7: Verify Deployment

- [ ] Check outputs
  ```bash
  terraform output
  ```

- [ ] Get specific output
  ```bash
  terraform output -json instance_public_ips | jq -r '.[]'
  ```

- [ ] Test web servers
  ```bash
  # Test each server
  for ip in $(terraform output -json instance_public_ips | jq -r '.[]'); do
    echo "Testing $ip"
    curl http://$ip
    echo ""
  done
  ```

- [ ] SSH into an instance
  ```bash
  ssh -i ~/.ssh/cicd-lab-key ubuntu@$(terraform output -json instance_public_ips | jq -r '.[0]')
  ```

#### Step 8: Export Infrastructure Data

- [ ] Create inventory file for Ansible
  ```bash
  cat > ../ansible/inventory/hosts.ini <<EOF
  [webservers]
  $(terraform output -json instance_public_ips | jq -r '.[]' | awk '{print $1" ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/cicd-lab-key"}')
  
  [webservers:vars]
  ansible_python_interpreter=/usr/bin/python3
  EOF
  
  cat ../ansible/inventory/hosts.ini
  ```

#### Step 9: Document Infrastructure

- [ ] Create diagram (optional)
  ```bash
  terraform graph | dot -Tpng > infrastructure-diagram.png
  ```

#### Step 10: Clean Up (Optional)

- [ ] Keep infrastructure running for next lab
- [ ] Or destroy if needed
  ```bash
  # terraform destroy
  ```

---

## Lab 3: Server Configuration with Ansible

### Theory: Configuration Management

**What is Ansible?**
- Agentless automation tool
- Uses SSH for communication
- YAML-based playbooks
- Idempotent by design
- Push-based model

**Ansible Concepts:**
- **Inventory**: List of managed hosts
- **Playbooks**: YAML files defining tasks
- **Tasks**: Individual configuration steps
- **Modules**: Pre-built functionality
- **Handlers**: Triggered tasks
- **Variables**: Dynamic values
- **Templates**: Jinja2 templated files

**Ansible in CI/CD:**
- Consistent server configuration
- Version-controlled configurations
- Repeatable deployments
- Environment-specific configs
- Integration with provisioning tools

### Practice: Configure Servers with Ansible

#### Step 1: Setup Ansible Directory

- [ ] Navigate to ansible directory
  ```bash
  cd ~/cicd-pipeline-lab/ansible
  ```

- [ ] Create directory structure
  ```bash
  mkdir -p {inventory,playbooks,roles,templates,files,group_vars,host_vars}
  ```

#### Step 2: Configure Ansible

- [ ] Create `ansible.cfg`
  ```ini
  [defaults]
  inventory = inventory/hosts.ini
  host_key_checking = False
  remote_user = ubuntu
  private_key_file = ~/.ssh/cicd-lab-key
  retry_files_enabled = False
  gathering = smart
  fact_caching = jsonfile
  fact_caching_connection = /tmp/ansible_facts
  fact_caching_timeout = 3600
  
  [privilege_escalation]
  become = True
  become_method = sudo
  become_user = root
  become_ask_pass = False
  
  [ssh_connection]
  ssh_args = -o ControlMaster=auto -o ControlPersist=60s
  pipelining = True
  ```

#### Step 3: Create Inventory

- [ ] Create `inventory/hosts.ini` (if not already created from Lab 2)
  ```ini
  [webservers]
  web1 ansible_host=YOUR_INSTANCE_1_IP ansible_user=ubuntu
  web2 ansible_host=YOUR_INSTANCE_2_IP ansible_user=ubuntu
  
  [webservers:vars]
  ansible_ssh_private_key_file=~/.ssh/cicd-lab-key
  ansible_python_interpreter=/usr/bin/python3
  
  [all:vars]
  environment=dev
  project_name=cicd-pipeline
  ```

- [ ] Or use dynamic inventory from Terraform
  ```bash
  cd ../lab2-infrastructure
  terraform output -json instance_public_ips | jq -r '.[]' > /tmp/ips.txt
  cd ../ansible
  
  cat > inventory/hosts.ini <<'EOF'
  [webservers]
  EOF
  
  i=1
  while read ip; do
    echo "web${i} ansible_host=${ip} ansible_user=ubuntu" >> inventory/hosts.ini
    i=$((i+1))
  done < /tmp/ips.txt
  
  cat >> inventory/hosts.ini <<'EOF'
  
  [webservers:vars]
  ansible_ssh_private_key_file=~/.ssh/cicd-lab-key
  ansible_python_interpreter=/usr/bin/python3
  EOF
  ```

#### Step 4: Test Connectivity

- [ ] Ping all hosts
  ```bash
  ansible all -m ping
  ```

- [ ] Expected output: SUCCESS for each host

- [ ] Gather facts
  ```bash
  ansible all -m setup -a "filter=ansible_distribution*"
  ```

- [ ] Run ad-hoc command
  ```bash
  ansible webservers -m command -a "uptime"
  ```

#### Step 5: Create Variables

- [ ] Create `group_vars/all.yml`
  ```yaml
  ---
  # Common variables for all hosts
  project_name: cicd-pipeline
  environment: dev
  
  # Application settings
  app_name: sample-app
  app_port: 3000
  app_user: webapp
  app_directory: "/opt/{{ app_name }}"
  
  # Nginx settings
  nginx_port: 80
  nginx_server_name: "_"
  
  # System packages
  system_packages:
    - curl
    - wget
    - git
    - htop
    - vim
    - net-tools
  ```

- [ ] Create `group_vars/webservers.yml`
  ```yaml
  ---
  # Variables specific to webservers
  web_packages:
    - nginx
    - nodejs
    - npm
  
  nginx_worker_processes: auto
  nginx_worker_connections: 1024
  ```

#### Step 6: Create Nginx Template

- [ ] Create `templates/nginx-site.conf.j2`
  ```nginx
  server {
      listen {{ nginx_port }};
      server_name {{ nginx_server_name }};
  
      access_log /var/log/nginx/{{ app_name }}_access.log;
      error_log /var/log/nginx/{{ app_name }}_error.log;
  
      location / {
          proxy_pass http://localhost:{{ app_port }};
          proxy_http_version 1.1;
          proxy_set_header Upgrade $http_upgrade;
          proxy_set_header Connection 'upgrade';
          proxy_set_header Host $host;
          proxy_set_header X-Real-IP $remote_addr;
          proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
          proxy_set_header X-Forwarded-Proto $scheme;
          proxy_cache_bypass $http_upgrade;
      }
  
      location /health {
          access_log off;
          return 200 "healthy\n";
          add_header Content-Type text/plain;
      }
  }
  ```

#### Step 7: Create Main Playbook

- [ ] Create `playbooks/configure-webservers.yml`
  ```yaml
  ---
  - name: Configure Web Servers
    hosts: webservers
    become: yes
    
    tasks:
      - name: Update apt cache
        apt:
          update_cache: yes
          cache_valid_time: 3600
        tags: ['packages']
      
      - name: Install system packages
        apt:
          name: "{{ system_packages }}"
          state: present
        tags: ['packages']
      
      - name: Install web server packages
        apt:
          name: "{{ web_packages }}"
          state: present
        tags: ['packages', 'web']
      
      - name: Create application user
        user:
          name: "{{ app_user }}"
          system: yes
          shell: /bin/bash
          home: "{{ app_directory }}"
          create_home: yes
        tags: ['users']
      
      - name: Create application directory
        file:
          path: "{{ app_directory }}"
          state: directory
          owner: "{{ app_user }}"
          group: "{{ app_user }}"
          mode: '0755'
        tags: ['directories']
      
      - name: Deploy Nginx configuration
        template:
          src: ../templates/nginx-site.conf.j2
          dest: "/etc/nginx/sites-available/{{ app_name }}"
          owner: root
          group: root
          mode: '0644'
        notify: Restart Nginx
        tags: ['nginx', 'config']
      
      - name: Enable Nginx site
        file:
          src: "/etc/nginx/sites-available/{{ app_name }}"
          dest: "/etc/nginx/sites-enabled/{{ app_name }}"
          state: link
        notify: Restart Nginx
        tags: ['nginx', 'config']
      
      - name: Remove default Nginx site
        file:
          path: /etc/nginx/sites-enabled/default
          state: absent
        notify: Restart Nginx
        tags: ['nginx', 'config']
      
      - name: Create sample Node.js application
        copy:
          content: |
            const http = require('http');
            const os = require('os');
            
            const server = http.createServer((req, res) => {
              if (req.url === '/health') {
                res.writeHead(200, {'Content-Type': 'text/plain'});
                res.end('healthy');
              } else {
                res.writeHead(200, {'Content-Type': 'text/html'});
                res.end(`
                  <h1>{{ project_name }} - {{ environment }}</h1>
                  <p>Hostname: ${os.hostname()}</p>
                  <p>Platform: ${os.platform()}</p>
                  <p>Node version: ${process.version}</p>
                  <p>Uptime: ${process.uptime()} seconds</p>
                `);
              }
            });
            
            server.listen({{ app_port }}, () => {
              console.log('Server running on port {{ app_port }}');
            });
          dest: "{{ app_directory }}/app.js"
          owner: "{{ app_user }}"
          group: "{{ app_user }}"
          mode: '0644'
        notify: Restart Application
        tags: ['app']
      
      - name: Create systemd service for application
        copy:
          content: |
            [Unit]
            Description={{ app_name }} Node.js Application
            After=network.target
            
            [Service]
            Type=simple
            User={{ app_user }}
            WorkingDirectory={{ app_directory }}
            ExecStart=/usr/bin/node {{ app_directory }}/app.js
            Restart=always
            RestartSec=10
            StandardOutput=syslog
            StandardError=syslog
            SyslogIdentifier={{ app_name }}
            
            [Install]
            WantedBy=multi-user.target
          dest: "/etc/systemd/system/{{ app_name }}.service"
          owner: root
          group: root
          mode: '0644'
        notify: Reload Systemd
        tags: ['app', 'systemd']
      
      - name: Enable and start application service
        systemd:
          name: "{{ app_name }}"
          enabled: yes
          state: started
          daemon_reload: yes
        tags: ['app', 'systemd']
      
      - name: Ensure Nginx is running
        systemd:
          name: nginx
          state: started
          enabled: yes
        tags: ['nginx']
      
      - name: Configure UFW firewall
        ufw:
          rule: allow
          port: "{{ item }}"
          proto: tcp
        loop:
          - '22'
          - '80'
          - '443'
        tags: ['security', 'firewall']
      
      - name: Enable UFW
        ufw:
          state: enabled
        tags: ['security', 'firewall']
    
    handlers:
      - name: Restart Nginx
        systemd:
          name: nginx
          state: restarted
      
      - name: Restart Application
        systemd:
          name: "{{ app_name }}"
          state: restarted
      
      - name: Reload Systemd
        systemd:
          daemon_reload: yes
  ```

#### Step 8: Run Playbook

- [ ] Check playbook syntax
  ```bash
  ansible-playbook playbooks/configure-webservers.yml --syntax-check
  ```

- [ ] Run in check mode (dry-run)
  ```bash
  ansible-playbook playbooks/configure-webservers.yml --check
  ```

- [ ] Run playbook
  ```bash
  ansible-playbook playbooks/configure-webservers.yml
  ```

- [ ] Run with specific tags
  ```bash
  ansible-playbook playbooks/configure-webservers.yml --tags "nginx,config"
  ```

- [ ] Run with verbose output
  ```bash
  ansible-playbook playbooks/configure-webservers.yml -v
  ```

#### Step 9: Verify Configuration

- [ ] Test web servers
  ```bash
  for host in $(grep ansible_host inventory/hosts.ini | awk '{print $2}' | cut -d= -f2); do
    echo "Testing $host"
    curl http://$host
    echo ""
  done
  ```

- [ ] Check application health
  ```bash
  for host in $(grep ansible_host inventory/hosts.ini | awk '{print $2}' | cut -d= -f2); do
    echo "Health check $host"
    curl http://$host/health
    echo ""
  done
  ```

- [ ] Check service status
  ```bash
  ansible webservers -m systemd -a "name=sample-app state=started" --become
  ```

#### Step 10: Create Additional Playbooks

- [ ] Create `playbooks/update-app.yml`
  ```yaml
  ---
  - name: Update Application
    hosts: webservers
    become: yes
    
    tasks:
      - name: Update application code
        copy:
          src: ../files/app.js
          dest: "{{ app_directory }}/app.js"
          owner: "{{ app_user }}"
          group: "{{ app_user }}"
          mode: '0644'
        notify: Restart Application
    
    handlers:
      - name: Restart Application
        systemd:
          name: "{{ app_name }}"
          state: restarted
  ```

- [ ] Create `playbooks/health-check.yml`
  ```yaml
  ---
  - name: Health Check
    hosts: webservers
    gather_facts: no
    
    tasks:
      - name: Check if application is responding
        uri:
          url: "http://{{ ansible_host }}/health"
          return_content: yes
        register: health_check
      
      - name: Display health status
        debug:
          msg: "{{ inventory_hostname }}: {{ health_check.content }}"
  ```

- [ ] Run health check
  ```bash
  ansible-playbook playbooks/health-check.yml
  ```

---

## Lab 4: Introduction to GitHub Actions

### Theory: CI/CD Automation with GitHub Actions

**What is GitHub Actions?**
- Native CI/CD platform integrated with GitHub
- Event-driven workflow automation
- YAML-based workflow definition
- Extensive action marketplace
- Free for public repositories

**Core Concepts:**
- **Workflows**: Automated processes
- **Jobs**: Group of steps
- **Steps**: Individual tasks
- **Actions**: Reusable units
- **Runners**: Execution environment
- **Events**: Triggers (push, PR, schedule)

**GitHub Actions Structure:**
```yaml
name: Workflow Name
on: [events]
jobs:
  job-name:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - run: command
```

**Benefits for CI/CD:**
- Automated testing and deployment
- Infrastructure provisioning
- Configuration management
- Integration with AWS, Terraform, Ansible
- Secrets management

### Practice: Create GitHub Actions Workflow

#### Step 1: Initialize Git Repository

- [ ] Navigate to project root
  ```bash
  cd ~/cicd-pipeline-lab
  ```

- [ ] Initialize Git repository
  ```bash
  git init
  ```

- [ ] Create `.gitignore`
  ```bash
  cat > .gitignore <<'EOF'
  # Terraform
  **/.terraform/*
  *.tfstate
  *.tfstate.*
  *.tfvars
  .terraform.lock.hcl
  tfplan
  
  # Ansible
  *.retry
  /tmp/ansible_facts/*
  
  # SSH Keys
  *.pem
  *.key
  
  # IDE
  .vscode/
  .idea/
  *.swp
  *.swo
  *~
  
  # OS
  .DS_Store
  Thumbs.db
  EOF
  ```

- [ ] Create initial commit
  ```bash
  git add .
  git commit -m "Initial commit: CI/CD pipeline infrastructure"
  ```

#### Step 2: Create GitHub Repository

- [ ] Create repository on GitHub
  ```bash
  # Using GitHub CLI
  gh repo create cicd-pipeline-lab --public --source=. --remote=origin
  
  # Or manually create on github.com and add remote
  # git remote add origin https://github.com/YOUR_USERNAME/cicd-pipeline-lab.git
  ```

- [ ] Push code
  ```bash
  git branch -M main
  git push -u origin main
  ```

#### Step 3: Create Workflow Directory

- [ ] Create GitHub Actions directory
  ```bash
  mkdir -p .github/workflows
  ```

#### Step 4: Create Simple Workflow

- [ ] Create `.github/workflows/test.yml`
  ```yaml
  name: Test Workflow
  
  on:
    push:
      branches:
        - main
    pull_request:
      branches:
        - main
  
  jobs:
    test-terraform:
      name: Test Terraform Configuration
      runs-on: ubuntu-latest
      
      steps:
        - name: Checkout code
          uses: actions/checkout@v3
        
        - name: Setup Terraform
          uses: hashicorp/setup-terraform@v2
          with:
            terraform_version: 1.6.0
        
        - name: Terraform Format Check
          working-directory: ./lab2-infrastructure
          run: terraform fmt -check -recursive
        
        - name: Terraform Init
          working-directory: ./lab2-infrastructure
          run: terraform init
        
        - name: Terraform Validate
          working-directory: ./lab2-infrastructure
          run: terraform validate
    
    test-ansible:
      name: Test Ansible Configuration
      runs-on: ubuntu-latest
      
      steps:
        - name: Checkout code
          uses: actions/checkout@v3
        
        - name: Setup Python
          uses: actions/setup-python@v4
          with:
            python-version: '3.10'
        
        - name: Install Ansible
          run: |
            pip install ansible
            ansible --version
        
        - name: Ansible Syntax Check
          working-directory: ./ansible
          run: |
            ansible-playbook playbooks/configure-webservers.yml --syntax-check
  ```

#### Step 5: Configure GitHub Secrets

- [ ] Navigate to repository settings on GitHub
- [ ] Go to Settings → Secrets and variables → Actions
- [ ] Add the following secrets:

**Required Secrets:**
```
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
SSH_PRIVATE_KEY
SSH_KEY_NAME
```

- [ ] Get AWS credentials
  ```bash
  aws configure get aws_access_key_id
  aws configure get aws_secret_access_key
  ```

- [ ] Get SSH private key
  ```bash
  cat ~/.ssh/cicd-lab-key
  ```

#### Step 6: Test Workflow

- [ ] Commit and push workflow
  ```bash
  git add .github/workflows/test.yml
  git commit -m "Add GitHub Actions test workflow"
  git push
  ```

- [ ] Navigate to Actions tab on GitHub
- [ ] Watch workflow execution
- [ ] Review logs

#### Step 7: Create Validation Workflow

- [ ] Create `.github/workflows/validate.yml`
  ```yaml
  name: Validate Configuration
  
  on:
    pull_request:
      branches:
        - main
  
  jobs:
    validate:
      name: Validate All Configurations
      runs-on: ubuntu-latest
      
      steps:
        - name: Checkout code
          uses: actions/checkout@v3
        
        - name: Setup Terraform
          uses: hashicorp/setup-terraform@v2
          with:
            terraform_version: 1.6.0
        
        - name: Setup Python
          uses: actions/setup-python@v4
          with:
            python-version: '3.10'
        
        - name: Install Ansible
          run: pip install ansible
        
        - name: Validate Terraform
          run: |
            cd lab2-infrastructure
            terraform init
            terraform validate
            terraform fmt -check
        
        - name: Validate Ansible
          run: |
            cd ansible
            ansible-playbook playbooks/configure-webservers.yml --syntax-check
            ansible-lint playbooks/*.yml || true
        
        - name: Security Scan
          run: |
            echo "Running security scans..."
            # Add security scanning tools here
  ```

#### Step 8: Create PR to Test

- [ ] Create new branch
  ```bash
  git checkout -b feature/test-validation
  ```

- [ ] Make a small change
  ```bash
  echo "# Test Change" >> README.md
  ```

- [ ] Commit and push
  ```bash
  git add README.md
  git commit -m "Test validation workflow"
  git push -u origin feature/test-validation
  ```

- [ ] Create Pull Request on GitHub
- [ ] Watch validation workflow run
- [ ] Merge if successful

#### Step 9: Understanding Workflow Triggers

- [ ] Create `.github/workflows/scheduled.yml`
  ```yaml
  name: Scheduled Health Check
  
  on:
    schedule:
      # Runs at 00:00 UTC every day
      - cron: '0 0 * * *'
    workflow_dispatch:  # Manual trigger
  
  jobs:
    health-check:
      name: Check Infrastructure Health
      runs-on: ubuntu-latest
      
      steps:
        - name: Checkout code
          uses: actions/checkout@v3
        
        - name: Configure AWS Credentials
          uses: aws-actions/configure-aws-credentials@v2
          with:
            aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
            aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
            aws-region: us-east-1
        
        - name: Check EC2 Instances
          run: |
            aws ec2 describe-instances \
              --filters "Name=tag:Project,Values=cicd-pipeline" \
              --query 'Reservations[*].Instances[*].[InstanceId,State.Name,PublicIpAddress]' \
              --output table
  ```

#### Step 10: Manual Workflow Trigger

- [ ] Commit workflow
  ```bash
  git add .github/workflows/scheduled.yml
  git commit -m "Add scheduled health check workflow"
  git push
  ```

- [ ] Go to Actions tab on GitHub
- [ ] Select "Scheduled Health Check"
- [ ] Click "Run workflow"
- [ ] Watch execution

---

## Lab 5: Complete CI/CD Pipeline

### Theory: End-to-End Automation

**Complete Pipeline Flow:**
1. Code push triggers workflow
2. Validation and testing
3. Infrastructure provisioning (Terraform)
4. Configuration management (Ansible)
5. Application deployment
6. Health checks
7. Notifications

**Pipeline Architecture:**
- Multiple jobs with dependencies
- Parallel execution where possible
- State passing between jobs
- Error handling and rollback
- Environment-specific deployments

**Best Practices:**
- Use job outputs for data passing
- Implement proper error handling
- Add approval gates for production
- Use caching to speed up builds
- Implement proper logging

### Practice: Build Complete Pipeline

#### Step 1: Create Complete Pipeline Workflow

- [ ] Create `.github/workflows/deploy.yml`
  ```yaml
  name: Complete CI/CD Pipeline
  
  on:
    push:
      branches:
        - main
    workflow_dispatch:
      inputs:
        environment:
          description: 'Environment to deploy'
          required: true
          default: 'dev'
          type: choice
          options:
            - dev
            - staging
            - prod
  
  env:
    AWS_REGION: us-east-1
    TF_VERSION: 1.6.0
    ANSIBLE_VERSION: 2.15.0
  
  jobs:
    validate:
      name: Validate Configuration
      runs-on: ubuntu-latest
      
      steps:
        - name: Checkout code
          uses: actions/checkout@v3
        
        - name: Setup Terraform
          uses: hashicorp/setup-terraform@v2
          with:
            terraform_version: ${{ env.TF_VERSION }}
        
        - name: Terraform Format Check
          working-directory: ./lab2-infrastructure
          run: terraform fmt -check -recursive
        
        - name: Terraform Validate
          working-directory: ./lab2-infrastructure
          run: |
            terraform init
            terraform validate
        
        - name: Setup Python
          uses: actions/setup-python@v4
          with:
            python-version: '3.10'
        
        - name: Install Ansible
          run: pip install ansible==${{ env.ANSIBLE_VERSION }}
        
        - name: Ansible Syntax Check
          working-directory: ./ansible
          run: |
            ansible-playbook playbooks/configure-webservers.yml --syntax-check
    
    terraform:
      name: Provision Infrastructure
      runs-on: ubuntu-latest
      needs: validate
      outputs:
        instance_ips: ${{ steps.tf-output.outputs.ips }}
        instance_count: ${{ steps.tf-output.outputs.count }}
      
      steps:
        - name: Checkout code
          uses: actions/checkout@v3
        
        - name: Configure AWS Credentials
          uses: aws-actions/configure-aws-credentials@v2
          with:
            aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
            aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
            aws-region: ${{ env.AWS_REGION }}
        
        - name: Setup Terraform
          uses: hashicorp/setup-terraform@v2
          with:
            terraform_version: ${{ env.TF_VERSION }}
            terraform_wrapper: false
        
        - name: Terraform Init
          working-directory: ./lab2-infrastructure
          run: terraform init
        
        - name: Terraform Plan
          working-directory: ./lab2-infrastructure
          run: |
            terraform plan \
              -var="ssh_key_name=${{ secrets.SSH_KEY_NAME }}" \
              -out=tfplan
        
        - name: Terraform Apply
          working-directory: ./lab2-infrastructure
          run: terraform apply -auto-approve tfplan
        
        - name: Get Terraform Outputs
          id: tf-output
          working-directory: ./lab2-infrastructure
          run: |
            IPS=$(terraform output -json instance_public_ips | jq -r 'join(",")')
            COUNT=$(terraform output -json instance_public_ips | jq 'length')
            echo "ips=$IPS" >> $GITHUB_OUTPUT
            echo "count=$COUNT" >> $GITHUB_OUTPUT
            echo "Instance IPs: $IPS"
            echo "Instance Count: $COUNT"
    
    ansible:
      name: Configure Servers
      runs-on: ubuntu-latest
      needs: terraform
      
      steps:
        - name: Checkout code
          uses: actions/checkout@v3
        
        - name: Setup Python
          uses: actions/setup-python@v4
          with:
            python-version: '3.10'
        
        - name: Install Ansible
          run: |
            pip install ansible==${{ env.ANSIBLE_VERSION }}
            ansible --version
        
        - name: Configure SSH Key
          run: |
            mkdir -p ~/.ssh
            echo "${{ secrets.SSH_PRIVATE_KEY }}" > ~/.ssh/id_rsa
            chmod 600 ~/.ssh/id_rsa
        
        - name: Create Dynamic Inventory
          working-directory: ./ansible
          run: |
            IFS=',' read -ra IPS <<< "${{ needs.terraform.outputs.instance_ips }}"
            
            cat > inventory/hosts.ini <<EOF
            [webservers]
            EOF
            
            i=1
            for ip in "${IPS[@]}"; do
              echo "web${i} ansible_host=${ip} ansible_user=ubuntu" >> inventory/hosts.ini
              ssh-keyscan -H ${ip} >> ~/.ssh/known_hosts
              i=$((i+1))
            done
            
            cat >> inventory/hosts.ini <<'EOF'
            
            [webservers:vars]
            ansible_ssh_private_key_file=~/.ssh/id_rsa
            ansible_python_interpreter=/usr/bin/python3
            EOF
            
            cat inventory/hosts.ini
        
        - name: Wait for Instances to be Ready
          run: |
            echo "Waiting for instances to initialize..."
            sleep 60
        
        - name: Test Connectivity
          working-directory: ./ansible
          run: |
            ansible all -m ping -v
        
        - name: Run Ansible Playbook
          working-directory: ./ansible
          run: |
            ansible-playbook playbooks/configure-webservers.yml -v
    
    verify:
      name: Verify Deployment
      runs-on: ubuntu-latest
      needs: [terraform, ansible]
      
      steps:
        - name: Checkout code
          uses: actions/checkout@v3
        
        - name: Setup Python
          uses: actions/setup-python@v4
          with:
            python-version: '3.10'
        
        - name: Install Ansible
          run: pip install ansible
        
        - name: Configure SSH Key
          run: |
            mkdir -p ~/.ssh
            echo "${{ secrets.SSH_PRIVATE_KEY }}" > ~/.ssh/id_rsa
            chmod 600 ~/.ssh/id_rsa
        
        - name: Create Inventory for Health Check
          working-directory: ./ansible
          run: |
            IFS=',' read -ra IPS <<< "${{ needs.terraform.outputs.instance_ips }}"
            
            cat > inventory/hosts.ini <<EOF
            [webservers]
            EOF
            
            i=1
            for ip in "${IPS[@]}"; do
              echo "web${i} ansible_host=${ip} ansible_user=ubuntu" >> inventory/hosts.ini
              ssh-keyscan -H ${ip} >> ~/.ssh/known_hosts
              i=$((i+1))
            done
            
            cat >> inventory/hosts.ini <<'EOF'
            
            [webservers:vars]
            ansible_ssh_private_key_file=~/.ssh/id_rsa
            ansible_python_interpreter=/usr/bin/python3
            EOF
        
        - name: Run Health Check
          working-directory: ./ansible
          run: |
            ansible-playbook playbooks/health-check.yml
        
        - name: Test HTTP Endpoints
          run: |
            IFS=',' read -ra IPS <<< "${{ needs.terraform.outputs.instance_ips }}"
            
            for ip in "${IPS[@]}"; do
              echo "Testing http://${ip}"
              curl -f http://${ip} || exit 1
              echo "Testing http://${ip}/health"
              curl -f http://${ip}/health || exit 1
            done
    
    notify:
      name: Send Notification
      runs-on: ubuntu-latest
      needs: [terraform, ansible, verify]
      if: always()
      
      steps:
        - name: Deployment Status
          run: |
            if [ "${{ needs.verify.result }}" == "success" ]; then
              echo "Deployment successful!"
              echo "Instance IPs: ${{ needs.terraform.outputs.instance_ips }}"
              echo "Instance Count: ${{ needs.terraform.outputs.instance_count }}"
            else
              echo "Deployment failed!"
              exit 1
            fi
  ```

#### Step 2: Test Complete Pipeline

- [ ] Commit and push
  ```bash
  git add .github/workflows/deploy.yml
  git commit -m "Add complete CI/CD pipeline"
  git push
  ```

- [ ] Navigate to Actions tab
- [ ] Watch complete pipeline execution
- [ ] Monitor each job
- [ ] Review logs

#### Step 3: Create Destroy Workflow

- [ ] Create `.github/workflows/destroy.yml`
  ```yaml
  name: Destroy Infrastructure
  
  on:
    workflow_dispatch:
      inputs:
        confirm:
          description: 'Type "destroy" to confirm'
          required: true
  
  jobs:
    destroy:
      name: Destroy Infrastructure
      runs-on: ubuntu-latest
      
      steps:
        - name: Validate Confirmation
          run: |
            if [ "${{ github.event.inputs.confirm }}" != "destroy" ]; then
              echo "Confirmation failed. Type 'destroy' to proceed."
              exit 1
            fi
        
        - name: Checkout code
          uses: actions/checkout@v3
        
        - name: Configure AWS Credentials
          uses: aws-actions/configure-aws-credentials@v2
          with:
            aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
            aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
            aws-region: us-east-1
        
        - name: Setup Terraform
          uses: hashicorp/setup-terraform@v2
          with:
            terraform_version: 1.6.0
        
        - name: Terraform Init
          working-directory: ./lab2-infrastructure
          run: terraform init
        
        - name: Terraform Destroy
          working-directory: ./lab2-infrastructure
          run: |
            terraform destroy \
              -var="ssh_key_name=${{ secrets.SSH_KEY_NAME }}" \
              -auto-approve
        
        - name: Confirm Destruction
          run: echo "Infrastructure destroyed successfully"
  ```

#### Step 4: Add Pipeline Status Badge

- [ ] Create `README.md` in project root
  ```markdown
  # CI/CD Pipeline Lab
  
  ![CI/CD Pipeline](https://github.com/YOUR_USERNAME/cicd-pipeline-lab/actions/workflows/deploy.yml/badge.svg)
  
  Complete CI/CD pipeline using Terraform, Ansible, and GitHub Actions.
  
  ## Architecture
  
  - **Terraform**: Infrastructure provisioning
  - **Ansible**: Configuration management
  - **GitHub Actions**: CI/CD orchestration
  
  ## Workflows
  
  - `deploy.yml`: Complete deployment pipeline
  - `test.yml`: Configuration validation
  - `destroy.yml`: Infrastructure teardown
  
  ## Usage
  
  1. Push to main branch to trigger deployment
  2. Manual trigger via Actions tab
  3. Destroy via workflow dispatch
  ```

- [ ] Commit and push
  ```bash
  git add README.md
  git commit -m "Add README with pipeline badge"
  git push
  ```

#### Step 5: Implement Environment-Specific Deployments

- [ ] Create `.github/workflows/deploy-env.yml`
  ```yaml
  name: Environment Deployment
  
  on:
    workflow_dispatch:
      inputs:
        environment:
          description: 'Environment'
          required: true
          type: choice
          options:
            - dev
            - staging
            - prod
  
  jobs:
    deploy:
      name: Deploy to ${{ github.event.inputs.environment }}
      runs-on: ubuntu-latest
      environment: ${{ github.event.inputs.environment }}
      
      steps:
        - name: Checkout code
          uses: actions/checkout@v3
        
        - name: Deploy to Environment
          run: |
            echo "Deploying to ${{ github.event.inputs.environment }}"
            # Add environment-specific deployment logic
  ```

#### Step 6: Monitor Pipeline Performance

- [ ] View workflow execution times
- [ ] Identify bottlenecks
- [ ] Review logs for optimization opportunities

---

## Lab 6: Advanced Pipeline Features

### Theory: Advanced CI/CD Concepts

**Advanced Features:**
- **Caching**: Speed up builds
- **Matrix Builds**: Test multiple configurations
- **Reusable Workflows**: DRY principle
- **Environments**: Approval gates
- **Conditional Execution**: Skip unnecessary steps
- **Artifacts**: Share data between jobs

**Security Considerations:**
- Least privilege access
- Secret rotation
- Environment protection rules
- Code scanning
- Dependency checking

**Monitoring & Observability:**
- Pipeline metrics
- Deployment tracking
- Error reporting
- Performance monitoring

### Practice: Implement Advanced Features

#### Step 1: Add Caching

- [ ] Create `.github/workflows/deploy-cached.yml`
  ```yaml
  name: Cached Deployment Pipeline
  
  on:
    push:
      branches:
        - main
  
  jobs:
    terraform:
      name: Terraform with Cache
      runs-on: ubuntu-latest
      
      steps:
        - name: Checkout code
          uses: actions/checkout@v3
        
        - name: Configure AWS Credentials
          uses: aws-actions/configure-aws-credentials@v2
          with:
            aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
            aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
            aws-region: us-east-1
        
        - name: Setup Terraform
          uses: hashicorp/setup-terraform@v2
          with:
            terraform_version: 1.6.0
        
        - name: Cache Terraform
          uses: actions/cache@v3
          with:
            path: |
              lab2-infrastructure/.terraform
              lab2-infrastructure/.terraform.lock.hcl
            key: terraform-${{ hashFiles('lab2-infrastructure/**/*.tf') }}
            restore-keys: |
              terraform-
        
        - name: Terraform Init
          working-directory: ./lab2-infrastructure
          run: terraform init
        
        - name: Terraform Plan
          working-directory: ./lab2-infrastructure
          run: terraform plan
    
    ansible:
      name: Ansible with Cache
      runs-on: ubuntu-latest
      
      steps:
        - name: Checkout code
          uses: actions/checkout@v3
        
        - name: Setup Python
          uses: actions/setup-python@v4
          with:
            python-version: '3.10'
        
        - name: Cache pip packages
          uses: actions/cache@v3
          with:
            path: ~/.cache/pip
            key: pip-${{ hashFiles('**/requirements.txt') }}
            restore-keys: |
              pip-
        
        - name: Install Ansible
          run: pip install ansible
  ```

#### Step 2: Implement Matrix Strategy

- [ ] Create `.github/workflows/matrix-test.yml`
  ```yaml
  name: Matrix Testing
  
  on:
    push:
      branches:
        - main
  
  jobs:
    test:
      name: Test on ${{ matrix.os }} with ${{ matrix.tf-version }}
      runs-on: ${{ matrix.os }}
      
      strategy:
        matrix:
          os: [ubuntu-latest, ubuntu-20.04]
          tf-version: [1.5.0, 1.6.0]
        fail-fast: false
      
      steps:
        - name: Checkout code
          uses: actions/checkout@v3
        
        - name: Setup Terraform ${{ matrix.tf-version }}
          uses: hashicorp/setup-terraform@v2
          with:
            terraform_version: ${{ matrix.tf-version }}
        
        - name: Terraform Version
          run: terraform version
        
        - name: Terraform Validate
          working-directory: ./lab2-infrastructure
          run: |
            terraform init
            terraform validate
  ```

#### Step 3: Create Reusable Workflow

- [ ] Create `.github/workflows/reusable-terraform.yml`
  ```yaml
  name: Reusable Terraform Workflow
  
  on:
    workflow_call:
      inputs:
        working-directory:
          required: true
          type: string
        terraform-version:
          required: false
          type: string
          default: '1.6.0'
      secrets:
        aws-access-key-id:
          required: true
        aws-secret-access-key:
          required: true
  
  jobs:
    terraform:
      name: Terraform Operations
      runs-on: ubuntu-latest
      
      steps:
        - name: Checkout code
          uses: actions/checkout@v3
        
        - name: Configure AWS Credentials
          uses: aws-actions/configure-aws-credentials@v2
          with:
            aws-access-key-id: ${{ secrets.aws-access-key-id }}
            aws-secret-access-key: ${{ secrets.aws-secret-access-key }}
            aws-region: us-east-1
        
        - name: Setup Terraform
          uses: hashicorp/setup-terraform@v2
          with:
            terraform_version: ${{ inputs.terraform-version }}
        
        - name: Terraform Init
          working-directory: ${{ inputs.working-directory }}
          run: terraform init
        
        - name: Terraform Plan
          working-directory: ${{ inputs.working-directory }}
          run: terraform plan
  ```

- [ ] Create `.github/workflows/use-reusable.yml`
  ```yaml
  name: Use Reusable Workflow
  
  on:
    workflow_dispatch:
  
  jobs:
    call-terraform:
      uses: ./.github/workflows/reusable-terraform.yml
      with:
        working-directory: ./lab2-infrastructure
        terraform-version: '1.6.0'
      secrets:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
  ```

#### Step 4: Implement Approval Gates

- [ ] Create environment in GitHub
  - Go to Settings → Environments
  - Create "production" environment
  - Add required reviewers
  - Set deployment branch to `main`

- [ ] Create `.github/workflows/deploy-with-approval.yml`
  ```yaml
  name: Deploy with Approval
  
  on:
    workflow_dispatch:
  
  jobs:
    deploy-staging:
      name: Deploy to Staging
      runs-on: ubuntu-latest
      environment: staging
      
      steps:
        - name: Deploy to Staging
          run: echo "Deploying to staging..."
    
    deploy-production:
      name: Deploy to Production
      runs-on: ubuntu-latest
      needs: deploy-staging
      environment: production
      
      steps:
        - name: Deploy to Production
          run: echo "Deploying to production..."
  ```

#### Step 5: Add Artifact Upload

- [ ] Create `.github/workflows/artifacts.yml`
  ```yaml
  name: Build and Upload Artifacts
  
  on:
    push:
      branches:
        - main
  
  jobs:
    build:
      name: Build Infrastructure Plan
      runs-on: ubuntu-latest
      
      steps:
        - name: Checkout code
          uses: actions/checkout@v3
        
        - name: Configure AWS Credentials
          uses: aws-actions/configure-aws-credentials@v2
          with:
            aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
            aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
            aws-region: us-east-1
        
        - name: Setup Terraform
          uses: hashicorp/setup-terraform@v2
          with:
            terraform_version: 1.6.0
        
        - name: Terraform Init and Plan
          working-directory: ./lab2-infrastructure
          run: |
            terraform init
            terraform plan -out=tfplan
        
        - name: Upload Terraform Plan
          uses: actions/upload-artifact@v3
          with:
            name: terraform-plan
            path: lab2-infrastructure/tfplan
            retention-days: 7
        
        - name: Generate Infrastructure Report
          working-directory: ./lab2-infrastructure
          run: |
            terraform show -json tfplan > plan-report.json
        
        - name: Upload Infrastructure Report
          uses: actions/upload-artifact@v3
          with:
            name: infrastructure-report
            path: lab2-infrastructure/plan-report.json
            retention-days: 30
    
    apply:
      name: Apply Infrastructure
      runs-on: ubuntu-latest
      needs: build
      
      steps:
        - name: Checkout code
          uses: actions/checkout@v3
        
        - name: Download Terraform Plan
          uses: actions/download-artifact@v3
          with:
            name: terraform-plan
            path: lab2-infrastructure/
        
        - name: Configure AWS Credentials
          uses: aws-actions/configure-aws-credentials@v2
          with:
            aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
            aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
            aws-region: us-east-1
        
        - name: Setup Terraform
          uses: hashicorp/setup-terraform@v2
          with:
            terraform_version: 1.6.0
        
        - name: Terraform Init
          working-directory: ./lab2-infrastructure
          run: terraform init
        
        - name: Terraform Apply
          working-directory: ./lab2-infrastructure
          run: terraform apply -auto-approve tfplan
  ```

#### Step 6: Implement Conditional Execution

- [ ] Create `.github/workflows/conditional.yml`
  ```yaml
  name: Conditional Execution
  
  on:
    push:
      branches:
        - main
      paths:
        - 'terraform/**'
        - 'ansible/**'
  
  jobs:
    check-changes:
      name: Check What Changed
      runs-on: ubuntu-latest
      outputs:
        terraform: ${{ steps.filter.outputs.terraform }}
        ansible: ${{ steps.filter.outputs.ansible }}
      
      steps:
        - name: Checkout code
          uses: actions/checkout@v3
        
        - name: Check file changes
          uses: dorny/paths-filter@v2
          id: filter
          with:
            filters: |
              terraform:
                - 'lab2-infrastructure/**'
              ansible:
                - 'ansible/**'
    
    terraform:
      name: Run Terraform
      runs-on: ubuntu-latest
      needs: check-changes
      if: needs.check-changes.outputs.terraform == 'true'
      
      steps:
        - name: Checkout code
          uses: actions/checkout@v3
        
        - name: Run Terraform
          run: echo "Running Terraform..."
    
    ansible:
      name: Run Ansible
      runs-on: ubuntu-latest
      needs: check-changes
      if: needs.check-changes.outputs.ansible == 'true'
      
      steps:
        - name: Checkout code
          uses: actions/checkout@v3
        
        - name: Run Ansible
          run: echo "Running Ansible..."
  ```

#### Step 7: Add Security Scanning

- [ ] Create `.github/workflows/security.yml`
  ```yaml
  name: Security Scanning
  
  on:
    push:
      branches:
        - main
    pull_request:
  
  jobs:
    tfsec:
      name: TFSec Security Scan
      runs-on: ubuntu-latest
      
      steps:
        - name: Checkout code
          uses: actions/checkout@v3
        
        - name: Run TFSec
          uses: aquasecurity/tfsec-action@v1.0.0
          with:
            working_directory: lab2-infrastructure
            soft_fail: true
    
    ansible-lint:
      name: Ansible Lint
      runs-on: ubuntu-latest
      
      steps:
        - name: Checkout code
          uses: actions/checkout@v3
        
        - name: Run Ansible Lint
          uses: ansible/ansible-lint-action@main
          with:
            path: ansible/playbooks/
  ```

---

## Lab 7: Monitoring and Troubleshooting

### Theory: Pipeline Observability

**Monitoring Components:**
- Pipeline execution metrics
- Infrastructure health
- Application performance
- Error tracking
- Deployment frequency
- Mean time to recovery (MTTR)

**Troubleshooting Strategies:**
- Read error messages carefully
- Check logs systematically
- Verify credentials and permissions
- Test components independently
- Use verbose/debug modes
- Review recent changes

**Common Issues:**
- Authentication failures
- State locking issues
- Network connectivity
- Resource quotas
- Syntax errors
- Dependency conflicts

### Practice: Implement Monitoring

#### Step 1: Create Health Check Playbook

- [ ] Create `ansible/playbooks/health-check.yml`
  ```yaml
  ---
  - name: Comprehensive Health Check
    hosts: webservers
    gather_facts: yes
    
    tasks:
      - name: Check system uptime
        command: uptime
        register: uptime_result
      
      - name: Display uptime
        debug:
          msg: "{{ inventory_hostname }}: {{ uptime_result.stdout }}"
      
      - name: Check disk usage
        shell: df -h | grep -v tmpfs
        register: disk_usage
      
      - name: Display disk usage
        debug:
          msg: "{{ disk_usage.stdout_lines }}"
      
      - name: Check memory usage
        shell: free -h
        register: memory_usage
      
      - name: Display memory usage
        debug:
          msg: "{{ memory_usage.stdout_lines }}"
      
      - name: Check if Nginx is running
        systemd:
          name: nginx
          state: started
        check_mode: yes
        register: nginx_status
      
      - name: Display Nginx status
        debug:
          msg: "Nginx is {{ 'running' if nginx_status.changed == false else 'not running' }}"
      
      - name: Check if application is running
        systemd:
          name: sample-app
          state: started
        check_mode: yes
        register: app_status
      
      - name: Display application status
        debug:
          msg: "Application is {{ 'running' if app_status.changed == false else 'not running' }}"
      
      - name: Test HTTP endpoint
        uri:
          url: "http://localhost/health"
          return_content: yes
        register: http_check
      
      - name: Display HTTP response
        debug:
          msg: "HTTP health check: {{ http_check.content }}"
      
      - name: Check for security updates
        shell: apt list --upgradable 2>/dev/null | grep -i security | wc -l
        register: security_updates
      
      - name: Display security updates
        debug:
          msg: "{{ security_updates.stdout }} security updates available"
  ```

#### Step 2: Create Monitoring Workflow

- [ ] Create `.github/workflows/monitor.yml`
  ```yaml
  name: Infrastructure Monitoring
  
  on:
    schedule:
      - cron: '*/30 * * * *'  # Every 30 minutes
    workflow_dispatch:
  
  jobs:
    health-check:
      name: Health Check
      runs-on: ubuntu-latest
      
      steps:
        - name: Checkout code
          uses: actions/checkout@v3
        
        - name: Configure AWS Credentials
          uses: aws-actions/configure-aws-credentials@v2
          with:
            aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
            aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
            aws-region: us-east-1
        
        - name: Check EC2 Instances Status
          id: ec2-check
          run: |
            instances=$(aws ec2 describe-instances \
              --filters "Name=tag:Project,Values=cicd-pipeline" "Name=instance-state-name,Values=running" \
              --query 'Reservations[*].Instances[*].[InstanceId,State.Name,PublicIpAddress]' \
              --output json)
            
            echo "instances=$instances" >> $GITHUB_OUTPUT
            echo "Instance Status:"
            echo "$instances" | jq -r '.[][] | @tsv'
        
        - name: Setup Python
          uses: actions/setup-python@v4
          with:
            python-version: '3.10'
        
        - name: Install Ansible
          run: pip install ansible
        
        - name: Configure SSH
          run: |
            mkdir -p ~/.ssh
            echo "${{ secrets.SSH_PRIVATE_KEY }}" > ~/.ssh/id_rsa
            chmod 600 ~/.ssh/id_rsa
        
        - name: Create Dynamic Inventory
          working-directory: ./ansible
          run: |
            ips=$(echo '${{ steps.ec2-check.outputs.instances }}' | jq -r '.[][] | .[2]')
            
            cat > inventory/hosts.ini <<EOF
            [webservers]
            EOF
            
            i=1
            for ip in $ips; do
              if [ ! -z "$ip" ]; then
                echo "web${i} ansible_host=${ip} ansible_user=ubuntu" >> inventory/hosts.ini
                ssh-keyscan -H ${ip} >> ~/.ssh/known_hosts 2>/dev/null
                i=$((i+1))
              fi
            done
            
            cat >> inventory/hosts.ini <<'EOF'
            
            [webservers:vars]
            ansible_ssh_private_key_file=~/.ssh/id_rsa
            ansible_python_interpreter=/usr/bin/python3
            EOF
        
        - name: Run Health Check Playbook
          working-directory: ./ansible
          run: |
            ansible-playbook playbooks/health-check.yml || true
        
        - name: Test HTTP Endpoints
          run: |
            ips=$(echo '${{ steps.ec2-check.outputs.instances }}' | jq -r '.[][] | .[2]')
            
            for ip in $ips; do
              if [ ! -z "$ip" ]; then
                echo "Testing http://${ip}"
                response=$(curl -s -o /dev/null -w "%{http_code}" http://${ip}/health)
                if [ "$response" == "200" ]; then
                  echo "${ip} is healthy"
                else
                  echo "${ip} returned status ${response}"
                fi
              fi
            done
  ```

#### Step 3: Create Troubleshooting Guide

- [ ] Create `TROUBLESHOOTING.md`
  ```markdown
  # CI/CD Pipeline Troubleshooting Guide
  
  ## Common Issues and Solutions
  
  ### 1. Terraform Issues
  
  #### State Lock Errors
  **Error:** `Error locking state: Error acquiring the state lock`
  **Solution:**
  ```bash
  # Force unlock (use with caution)
  terraform force-unlock LOCK_ID
  ```
  
  #### Authentication Errors
  **Error:** `Error: error configuring Terraform AWS Provider`
  **Solution:**
  - Verify AWS credentials in GitHub Secrets
  - Check IAM permissions
  - Ensure credentials are not expired
  
  #### Resource Already Exists
  **Error:** `Error: resource already exists`
  **Solution:**
  ```bash
  # Import existing resource
  terraform import aws_instance.web i-1234567890abcdef0
  ```
  
  ### 2. Ansible Issues
  
  #### SSH Connection Failures
  **Error:** `Failed to connect to the host via ssh`
  **Solution:**
  - Verify SSH key is correct
  - Check security group allows port 22
  - Ensure instance is running
  - Add host to known_hosts:
  ```bash
  ssh-keyscan -H INSTANCE_IP >> ~/.ssh/known_hosts
  ```
  
  #### Permission Denied
  **Error:** `Permission denied (publickey)`
  **Solution:**
  - Check SSH key permissions: `chmod 600 ~/.ssh/key.pem`
  - Verify correct username (ubuntu for Ubuntu, ec2-user for Amazon Linux)
  - Confirm key matches what's in AWS
  
  #### Module Not Found
  **Error:** `The module XYZ was not found`
  **Solution:**
  ```bash
  # Update Ansible
  pip install --upgrade ansible
  
  # Install specific collection
  ansible-galaxy collection install community.general
  ```
  
  ### 3. GitHub Actions Issues
  
  #### Secrets Not Found
  **Error:** `Error: Required secret not found`
  **Solution:**
  - Navigate to Settings → Secrets → Actions
  - Verify secret name matches exactly
  - Re-add secret if necessary
  
  #### Workflow Not Triggering
  **Solution:**
  - Check workflow file is in `.github/workflows/`
  - Verify YAML syntax
  - Ensure trigger conditions are met
  - Check branch filters
  
  #### Job Timeout
  **Error:** `The job running on runner has exceeded the maximum execution time`
  **Solution:**
  ```yaml
  jobs:
    job-name:
      timeout-minutes: 60  # Increase timeout
  ```
  
  ### 4. AWS Issues
  
  #### Resource Limits
  **Error:** `You have exceeded your maximum vCPU limit`
  **Solution:**
  - Request limit increase in AWS Service Quotas
  - Use smaller instance types
  - Clean up unused resources
  
  #### Region Not Available
  **Error:** `The availability zone is not available`
  **Solution:**
  - Change to different availability zone
  - Use data source to get available zones:
  ```hcl
  data "aws_availability_zones" "available" {
    state = "available"
  }
  ```
  
  ## Debugging Commands
  
  ### Terraform Debug
  ```bash
  # Enable debug logging
  export TF_LOG=DEBUG
  export TF_LOG_PATH=./terraform-debug.log
  terraform apply
  
  # Validate configuration
  terraform validate
  
  # Format check
  terraform fmt -check -recursive
  
  # Show current state
  terraform show
  
  # List resources
  terraform state list
  ```
  
  ### Ansible Debug
  ```bash
  # Run with verbose output
  ansible-playbook playbook.yml -vvv
  
  # Check syntax
  ansible-playbook playbook.yml --syntax-check
  
  # Dry run
  ansible-playbook playbook.yml --check
  
  # Test connectivity
  ansible all -m ping -vvv
  
  # Gather facts
  ansible all -m setup
  ```
  
  ### GitHub Actions Debug
  ```bash
  # Enable debug logging
  # Add secrets:
  # ACTIONS_STEP_DEBUG = true
  # ACTIONS_RUNNER_DEBUG = true
  
  # View workflow runs
  gh run list
  
  # View specific run
  gh run view RUN_ID
  
  # Download logs
  gh run download RUN_ID
  ```
  
  ## Monitoring Commands
  
  ### Check Infrastructure
  ```bash
  # List EC2 instances
  aws ec2 describe-instances \
    --filters "Name=tag:Project,Values=cicd-pipeline" \
    --query 'Reservations[*].Instances[*].[InstanceId,State.Name,PublicIpAddress]'
  
  # Check security groups
  aws ec2 describe-security-groups \
    --filters "Name=tag:Project,Values=cicd-pipeline"
  ```
  
  ### Check Application
  ```bash
  # Test HTTP endpoint
  curl -v http://INSTANCE_IP
  
  # Test health endpoint
  curl http://INSTANCE_IP/health
  
  # Check from all instances
  for ip in IP1 IP2; do curl -s http://$ip; done
  ```
  
  ### SSH into Instance
  ```bash
  ssh -i ~/.ssh/cicd-lab-key ubuntu@INSTANCE_IP
  
  # Check logs
  sudo journalctl -u sample-app -f
  sudo tail -f /var/log/nginx/access.log
  
  # Check service status
  sudo systemctl status sample-app
  sudo systemctl status nginx
  ```
  
  ## Getting Help
  
  1. Check workflow logs in GitHub Actions
  2. Review Terraform/Ansible output
  3. Check AWS CloudWatch logs
  4. Consult official documentation
  5. Search GitHub Issues
  ```

#### Step 4: Add Logging Workflow

- [ ] Create `.github/workflows/logging.yml`
  ```yaml
  name: Enhanced Logging
  
  on:
    workflow_dispatch:
  
  jobs:
    deploy-with-logging:
      name: Deploy with Detailed Logs
      runs-on: ubuntu-latest
      
      steps:
        - name: Checkout code
          uses: actions/checkout@v3
        
        - name: Configure AWS Credentials
          uses: aws-actions/configure-aws-credentials@v2
          with:
            aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
            aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
            aws-region: us-east-1
        
        - name: Setup Terraform
          uses: hashicorp/setup-terraform@v2
          with:
            terraform_version: 1.6.0
        
        - name: Terraform Init with Logs
          working-directory: ./lab2-infrastructure
          run: |
            echo "::group::Terraform Init"
            terraform init
            echo "::endgroup::"
        
        - name: Terraform Plan with Logs
          working-directory: ./lab2-infrastructure
          run: |
            echo "::group::Terraform Plan"
            terraform plan -no-color | tee plan.log
            echo "::endgroup::"
        
        - name: Upload Plan Log
          uses: actions/upload-artifact@v3
          with:
            name: terraform-plan-log
            path: lab2-infrastructure/plan.log
        
        - name: Set Debug Output
          run: |
            echo "::debug::This is a debug message"
            echo "::notice::This is a notice"
            echo "::warning::This is a warning"
  ```

#### Step 5: Test Monitoring

- [ ] Commit all changes
  ```bash
  git add .
  git commit -m "Add monitoring and troubleshooting"
  git push
  ```

- [ ] Manually trigger monitoring workflow
- [ ] Review outputs
- [ ] Test troubleshooting scenarios

---

## Best Practices and Next Steps

### CI/CD Best Practices Checklist

#### Pipeline Design
- [ ] Keep pipelines fast (< 10 minutes ideal)
- [ ] Fail fast (run quick tests first)
- [ ] Use parallel execution where possible
- [ ] Implement proper error handling
- [ ] Add retry logic for flaky operations
- [ ] Use caching to speed up builds
- [ ] Keep workflows DRY (use reusable workflows)

#### Security
- [ ] Never commit secrets to repository
- [ ] Rotate credentials regularly
- [ ] Use least privilege IAM policies
- [ ] Scan for security vulnerabilities
- [ ] Enable branch protection rules
- [ ] Require code reviews
- [ ] Use environment protection rules for production
- [ ] Implement approval gates for critical deployments

#### Terraform Best Practices
- [ ] Always use remote state with locking
- [ ] Use consistent naming conventions
- [ ] Tag all resources appropriately
- [ ] Use variables for flexibility
- [ ] Output useful information
- [ ] Document your modules
- [ ] Use `terraform fmt` consistently
- [ ] Validate before applying
- [ ] Plan before every apply

#### Ansible Best Practices
- [ ] Use roles for organization
- [ ] Keep playbooks idempotent
- [ ] Use variables for flexibility
- [ ] Use templates for configuration files
- [ ] Implement proper error handling
- [ ] Use tags for selective execution
- [ ] Keep secrets in vault
- [ ] Document playbook requirements
- [ ] Test playbooks before production

#### GitHub Actions Best Practices
- [ ] Use descriptive job and step names
- [ ] Group related steps
- [ ] Use conditional execution
- [ ] Implement proper logging
- [ ] Upload artifacts for debugging
- [ ] Use matrix builds for testing
- [ ] Set appropriate timeouts
- [ ] Monitor workflow performance

#### Monitoring and Observability
- [ ] Implement health checks
- [ ] Track deployment frequency
- [ ] Monitor failure rates
- [ ] Measure recovery time
- [ ] Log all significant events
- [ ] Set up alerting
- [ ] Create dashboards
- [ ] Document runbooks

### Common Commands Reference

#### Terraform Commands
```bash
# Initialize
terraform init
terraform init -upgrade

# Validate
terraform validate
terraform fmt -check
terraform fmt -recursive

# Plan
terraform plan
terraform plan -out=tfplan
terraform plan -var="key=value"

# Apply
terraform apply
terraform apply tfplan
terraform apply -auto-approve

# Destroy
terraform destroy
terraform destroy -target=resource

# State Management
terraform state list
terraform state show resource
terraform state rm resource
terraform state mv source destination
terraform state pull
terraform state push

# Workspace
terraform workspace list
terraform workspace new name
terraform workspace select name

# Output
terraform output
terraform output -json

# Import
terraform import resource id

# Graph
terraform graph | dot -Tpng > graph.png
```

#### Ansible Commands
```bash
# Ad-hoc Commands
ansible all -m ping
ansible all -m command -a "uptime"
ansible all -m setup

# Playbook Execution
ansible-playbook playbook.yml
ansible-playbook playbook.yml --check
ansible-playbook playbook.yml --diff
ansible-playbook playbook.yml -v
ansible-playbook playbook.yml --tags "tag1,tag2"
ansible-playbook playbook.yml --skip-tags "tag"
ansible-playbook playbook.yml --limit "host1,host2"

# Syntax Check
ansible-playbook playbook.yml --syntax-check

# List Tasks
ansible-playbook playbook.yml --list-tasks

# List Hosts
ansible-playbook playbook.yml --list-hosts

# Galaxy
ansible-galaxy collection install name
ansible-galaxy role install name

# Inventory
ansible-inventory --list
ansible-inventory --graph

# Vault
ansible-vault create file.yml
ansible-vault edit file.yml
ansible-vault encrypt file.yml
ansible-vault decrypt file.yml
```

#### GitHub CLI Commands
```bash
# Authentication
gh auth login
gh auth status

# Repository
gh repo create
gh repo view
gh repo clone

# Workflow
gh workflow list
gh workflow run workflow.yml
gh workflow view workflow.yml

# Run
gh run list
gh run view RUN_ID
gh run watch RUN_ID
gh run download RUN_ID

# Actions
gh actions --help

# Secrets
gh secret list
gh secret set SECRET_NAME
```

#### AWS CLI Commands
```bash
# EC2
aws ec2 describe-instances
aws ec2 start-instances --instance-ids i-xxx
aws ec2 stop-instances --instance-ids i-xxx
aws ec2 terminate-instances --instance-ids i-xxx

# S3
aws s3 ls
aws s3 cp file s3://bucket/
aws s3 sync . s3://bucket/

# IAM
aws iam list-users
aws iam get-user

# CloudWatch Logs
aws logs tail /aws/lambda/function --follow
```

### Next Steps in Your CI/CD Journey

#### Advanced Topics to Explore
- [ ] Blue-Green Deployments
- [ ] Canary Releases
- [ ] Feature Flags
- [ ] GitOps with ArgoCD or Flux
- [ ] Container Orchestration (Kubernetes)
- [ ] Service Mesh (Istio, Linkerd)
- [ ] Infrastructure Testing (Terratest, InSpec)
- [ ] Policy as Code (OPA, Sentinel)
- [ ] Secret Management (Vault, AWS Secrets Manager)
- [ ] Observability Stack (Prometheus, Grafana)

#### Learning Resources
- [ ] [Terraform Documentation](https://www.terraform.io/docs)
- [ ] [Ansible Documentation](https://docs.ansible.com)
- [ ] [GitHub Actions Docs](https://docs.github.com/actions)
- [ ] [AWS Well-Architected Framework](https://aws.amazon.com/architecture/well-architected)
- [ ] [The Twelve-Factor App](https://12factor.net)
- [ ] [GitOps Principles](https://www.gitops.tech)

#### Practice Projects
- [ ] Multi-region deployment
- [ ] Microservices architecture
- [ ] Kubernetes cluster with Terraform
- [ ] Complete observability stack
- [ ] Automated disaster recovery
- [ ] Compliance automation
- [ ] Cost optimization pipeline

### Conclusion

Congratulations! You've completed the CI/CD Pipeline Laboratory.

**You now have hands-on experience with:**
- Infrastructure provisioning with Terraform
- Configuration management with Ansible
- CI/CD automation with GitHub Actions
- Integration of all three tools
- Security best practices
- Monitoring and troubleshooting
- Advanced pipeline features

**Key Takeaways:**
- CI/CD pipelines automate infrastructure and deployments
- Infrastructure as Code ensures reproducibility
- Configuration management maintains consistency
- Automation reduces human error
- Monitoring is essential for reliability
- Security must be built into the pipeline

Keep practicing and building more complex pipelines. The skills you've learned are applicable to any cloud provider and can scale to enterprise-level deployments.

Kirk Patrick

---

**Lab Version:** 1.9
**Last Updated:** 2025  
**Tools Used:**
- Terraform 1.6.0+
- Ansible 2.15.0+
- GitHub Actions
- AWS Provider 5.0+
