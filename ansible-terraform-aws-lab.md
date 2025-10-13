# DevOps Advamced - Ansible with Terraform and AWS - Hands-On Lab

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [Lab 1: Environment Setup](#lab-1-environment-setup)
3. [Lab 2: Terraform Basics](#lab-2-terraform-basics)
4. [Lab 3: Ansible Basics](#lab-3-ansible-basics)
5. [Lab 4: Provisioning AWS Infrastructure with Terraform](#lab-4-provisioning-aws-infrastructure-with-terraform)
6. [Lab 5: Configuring AWS Resources with Ansible](#lab-5-configuring-aws-resources-with-ansible)
7. [Lab 6: Complete Integration Project](#lab-6-complete-integration-project)
8. [Lab 7: Advanced Topics](#lab-7-advanced-topics)

---

## Prerequisites

### Theory: Understanding the Tools

**Terraform** is an Infrastructure as Code (IaC) tool that allows you to define and provision cloud infrastructure using declarative configuration files. It's ideal for:
- Creating cloud resources (EC2, VPC, S3, etc.)
- Managing infrastructure lifecycle
- Versioning infrastructure changes

**Ansible** is a configuration management and automation tool that uses a simple, agentless approach. It's ideal for:
- Configuring servers and applications
- Deploying software
- Orchestrating complex workflows

**AWS (Amazon Web Services)** is a cloud computing platform providing various services like compute, storage, databases, and networking.

**Integration Strategy**: Terraform provisions the infrastructure, Ansible configures it.

### Checklist
- [ ] Have an AWS account with administrative access
- [ ] Have AWS CLI installed or ready to install
- [ ] Have a Linux/MacOS system or Windows with WSL2
- [ ] Basic understanding of command line
- [ ] Text editor installed (VS Code, Vim, or similar)

---

## Lab 1: Environment Setup

### Theory: Setting Up Your Workspace

Before working with infrastructure automation, you need a properly configured development environment. This includes:
- Installing necessary tools
- Configuring AWS credentials
- Setting up SSH keys for secure communication

### Tasks

#### 1.1 Install Terraform

**Linux/MacOS:**

```bash
# Download Terraform
wget https://releases.hashicorp.com/terraform/1.6.0/terraform_1.6.0_linux_amd64.zip

# Unzip and move to PATH
unzip terraform_1.6.0_linux_amd64.zip
sudo mv terraform /usr/local/bin/

# Verify installation
terraform version
```

**Windows (PowerShell):**

```powershell
# Using Chocolatey
choco install terraform

# Verify installation
terraform version
```

- [ ] Terraform installed successfully
- [ ] `terraform version` command works

#### 1.2 Install Ansible

**Linux/MacOS:**

```bash
# Ubuntu/Debian
sudo apt update
sudo apt install ansible -y

# MacOS
brew install ansible

# Verify installation
ansible --version
```

**Windows:**

```powershell
# Use WSL2 with Ubuntu and follow Linux instructions
```

- [ ] Ansible installed successfully
- [ ] `ansible --version` command works

#### 1.3 Install and Configure AWS CLI

```bash
# Linux
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install

# MacOS
brew install awscli

# Verify installation
aws --version
```

**Configure AWS credentials:**

```bash
aws configure
```

Enter your:
- AWS Access Key ID
- AWS Secret Access Key
- Default region (e.g., us-east-1)
- Default output format (json)

- [ ] AWS CLI installed
- [ ] AWS credentials configured
- [ ] `aws sts get-caller-identity` returns your account info

#### 1.4 Create Project Directory Structure

```bash
mkdir -p ~/ansible-terraform-lab
cd ~/ansible-terraform-lab
mkdir -p terraform ansible inventory
```

- [ ] Project directories created
- [ ] Navigated to project root

#### 1.5 Generate SSH Key Pair

```bash
ssh-keygen -t rsa -b 4096 -f ~/.ssh/aws-lab-key -N ""
chmod 400 ~/.ssh/aws-lab-key
```

- [ ] SSH key pair generated
- [ ] Private key permissions set to 400

---

## Lab 2: Terraform Basics

### Theory: Infrastructure as Code with Terraform

Terraform uses HCL (HashiCorp Configuration Language) to define infrastructure. Key concepts:

- **Providers**: Plugins that interact with cloud platforms (AWS, Azure, etc.)
- **Resources**: Infrastructure components (EC2 instances, S3 buckets, etc.)
- **Variables**: Parameterize configurations
- **Outputs**: Extract information from resources
- **State**: Tracks the current infrastructure state

### Tasks

#### 2.1 Create Your First Terraform Configuration

Create `terraform/main.tf`:

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

resource "aws_s3_bucket" "lab_bucket" {
  bucket = "ansible-terraform-lab-${random_id.bucket_id.hex}"
  
  tags = {
    Name        = "Lab Bucket"
    Environment = "Learning"
  }
}

resource "random_id" "bucket_id" {
  byte_length = 4
}

output "bucket_name" {
  value = aws_s3_bucket.lab_bucket.id
}
```

- [ ] `main.tf` file created
- [ ] Configuration syntax reviewed

#### 2.2 Initialize Terraform

```bash
cd ~/ansible-terraform-lab/terraform
terraform init
```

This downloads the AWS provider plugin.

- [ ] `terraform init` executed successfully
- [ ] `.terraform` directory created

#### 2.3 Plan Infrastructure Changes

```bash
terraform plan
```

Review the execution plan showing what will be created.

- [ ] `terraform plan` executed
- [ ] Plan output reviewed

#### 2.4 Apply Configuration

```bash
terraform apply
```

Type `yes` when prompted.

- [ ] `terraform apply` executed successfully
- [ ] S3 bucket created
- [ ] Bucket name noted from output

#### 2.5 Verify Resource Creation

```bash
aws s3 ls | grep ansible-terraform-lab
```

- [ ] S3 bucket verified in AWS
- [ ] Terraform state file (`terraform.tfstate`) exists

#### 2.6 Destroy Resources

```bash
terraform destroy
```

Type `yes` when prompted.

- [ ] Resources destroyed successfully
- [ ] S3 bucket removed from AWS

---

## Lab 3: Ansible Basics

### Theory: Configuration Management with Ansible

Ansible uses YAML-based playbooks to define automation tasks. Key concepts:

- **Inventory**: List of hosts to manage
- **Playbooks**: YAML files defining tasks
- **Modules**: Reusable units of work (e.g., copy files, install packages)
- **Tasks**: Individual actions to perform
- **Roles**: Organized collections of playbooks, variables, and files
- **Idempotency**: Running the same task multiple times produces the same result

### Tasks

#### 3.1 Create Local Inventory

Create `inventory/local.ini`:

```ini
[local]
localhost ansible_connection=local
```

- [ ] Inventory file created

#### 3.2 Test Ansible Connection

```bash
cd ~/ansible-terraform-lab
ansible -i inventory/local.ini local -m ping
```

Expected output: `localhost | SUCCESS`

- [ ] Ansible ping successful

#### 3.3 Create Your First Playbook

Create `ansible/first-playbook.yml`:

```yaml
---
- name: My First Ansible Playbook
  hosts: local
  gather_facts: yes
  
  tasks:
    - name: Display OS information
      debug:
        msg: "Running on {{ ansible_distribution }} {{ ansible_distribution_version }}"
    
    - name: Create a directory
      file:
        path: /tmp/ansible-lab
        state: directory
        mode: '0755'
    
    - name: Create a test file
      copy:
        content: "Hello from Ansible!\nTimestamp: {{ ansible_date_time.iso8601 }}"
        dest: /tmp/ansible-lab/test.txt
    
    - name: Display file contents
      command: cat /tmp/ansible-lab/test.txt
      register: file_contents
    
    - name: Show file contents
      debug:
        var: file_contents.stdout_lines
```

- [ ] Playbook created
- [ ] YAML syntax reviewed

#### 3.4 Run the Playbook

```bash
ansible-playbook -i inventory/local.ini ansible/first-playbook.yml
```

- [ ] Playbook executed successfully
- [ ] All tasks completed
- [ ] File created at `/tmp/ansible-lab/test.txt`

#### 3.5 Verify Idempotency

Run the playbook again:

```bash
ansible-playbook -i inventory/local.ini ansible/first-playbook.yml
```

Notice that most tasks show "ok" instead of "changed".

- [ ] Second run completed
- [ ] Observed idempotent behavior

---

## Lab 4: Provisioning AWS Infrastructure with Terraform

### Theory: Building AWS Infrastructure

In this lab, you'll create a complete AWS infrastructure including:
- VPC (Virtual Private Cloud) with public subnet
- Security Group with proper rules
- EC2 instance for application hosting
- Dynamic inventory for Ansible

### Tasks

#### 4.1 Create VPC and Networking Configuration

Create `terraform/vpc.tf`:

```hcl
resource "aws_vpc" "lab_vpc" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_hostnames = true
  enable_dns_support   = true
  
  tags = {
    Name = "ansible-terraform-lab-vpc"
  }
}

resource "aws_subnet" "public_subnet" {
  vpc_id                  = aws_vpc.lab_vpc.id
  cidr_block              = "10.0.1.0/24"
  availability_zone       = "us-east-1a"
  map_public_ip_on_launch = true
  
  tags = {
    Name = "ansible-terraform-lab-public-subnet"
  }
}

resource "aws_internet_gateway" "igw" {
  vpc_id = aws_vpc.lab_vpc.id
  
  tags = {
    Name = "ansible-terraform-lab-igw"
  }
}

resource "aws_route_table" "public_rt" {
  vpc_id = aws_vpc.lab_vpc.id
  
  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.igw.id
  }
  
  tags = {
    Name = "ansible-terraform-lab-public-rt"
  }
}

resource "aws_route_table_association" "public_rta" {
  subnet_id      = aws_subnet.public_subnet.id
  route_table_id = aws_route_table.public_rt.id
}
```

- [ ] VPC configuration created
- [ ] Networking components defined

#### 4.2 Create Security Group

Create `terraform/security.tf`:

```hcl
resource "aws_security_group" "web_sg" {
  name        = "ansible-terraform-lab-web-sg"
  description = "Security group for web server"
  vpc_id      = aws_vpc.lab_vpc.id
  
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
  
  ingress {
    description = "HTTPS"
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
    Name = "ansible-terraform-lab-web-sg"
  }
}
```

- [ ] Security group configuration created

#### 4.3 Create EC2 Instance Configuration

Create `terraform/ec2.tf`:

```hcl
data "aws_ami" "ubuntu" {
  most_recent = true
  owners      = ["099720109477"] # Canonical
  
  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-jammy-22.04-amd64-server-*"]
  }
}

resource "aws_key_pair" "lab_key" {
  key_name   = "ansible-terraform-lab-key"
  public_key = file("~/.ssh/aws-lab-key.pub")
}

resource "aws_instance" "web_server" {
  ami                    = data.aws_ami.ubuntu.id
  instance_type          = "t2.micro"
  subnet_id              = aws_subnet.public_subnet.id
  vpc_security_group_ids = [aws_security_group.web_sg.id]
  key_name               = aws_key_pair.lab_key.key_name
  
  user_data = <<-EOF
              #!/bin/bash
              apt-get update
              apt-get install -y python3 python3-pip
              EOF
  
  tags = {
    Name = "ansible-terraform-lab-web"
  }
}
```

- [ ] EC2 configuration created
- [ ] AMI data source defined

#### 4.4 Create Outputs

Create `terraform/outputs.tf`:

```hcl
output "instance_public_ip" {
  description = "Public IP of the EC2 instance"
  value       = aws_instance.web_server.public_ip
}

output "instance_id" {
  description = "ID of the EC2 instance"
  value       = aws_instance.web_server.id
}

output "vpc_id" {
  description = "ID of the VPC"
  value       = aws_vpc.lab_vpc.id
}
```

- [ ] Outputs configuration created

#### 4.5 Apply the Complete Infrastructure

```bash
cd ~/ansible-terraform-lab/terraform
terraform init
terraform plan
terraform apply
```

Save the output IP address!

- [ ] Infrastructure provisioned successfully
- [ ] EC2 instance public IP noted
- [ ] Can see instance in AWS Console

#### 4.6 Test SSH Connection

Wait 2-3 minutes for the instance to initialize, then:

```bash
ssh -i ~/.ssh/aws-lab-key ubuntu@<INSTANCE_PUBLIC_IP>
```

Type `yes` to accept the fingerprint, then `exit` to close the connection.

- [ ] SSH connection successful
- [ ] Can log into the instance

---

## Lab 5: Configuring AWS Resources with Ansible

### Theory: Managing EC2 Instances with Ansible

Now that infrastructure is provisioned, we'll use Ansible to:
- Configure the operating system
- Install and configure web servers
- Deploy applications
- Manage services

### Tasks

#### 5.1 Create Dynamic Inventory

Create `inventory/aws_ec2.yml`:

```yaml
plugin: amazon.aws.aws_ec2
regions:
  - us-east-1
filters:
  tag:Name: ansible-terraform-lab-web
  instance-state-name: running
hostnames:
  - public-ip-address
compose:
  ansible_host: public_ip_address
```

- [ ] Dynamic inventory configuration created

#### 5.2 Install Ansible AWS Collection

```bash
ansible-galaxy collection install amazon.aws
pip3 install boto3 botocore
```

- [ ] AWS collection installed
- [ ] Python dependencies installed

#### 5.3 Test Dynamic Inventory

```bash
cd ~/ansible-terraform-lab
ansible-inventory -i inventory/aws_ec2.yml --graph
```

You should see your EC2 instance listed.

- [ ] Dynamic inventory working
- [ ] EC2 instance visible in inventory

#### 5.4 Create Ansible Configuration

Create `ansible/ansible.cfg`:

```ini
[defaults]
host_key_checking = False
remote_user = ubuntu
private_key_file = ~/.ssh/aws-lab-key
inventory = ../inventory/aws_ec2.yml

[inventory]
enable_plugins = amazon.aws.aws_ec2
```

- [ ] Ansible configuration created

#### 5.5 Create Web Server Playbook

Create `ansible/webserver-setup.yml`:

```yaml
---
- name: Configure Web Server
  hosts: aws_ec2
  become: yes
  gather_facts: yes
  
  tasks:
    - name: Update apt cache
      apt:
        update_cache: yes
        cache_valid_time: 3600
    
    - name: Install Nginx
      apt:
        name: nginx
        state: present
    
    - name: Start and enable Nginx
      service:
        name: nginx
        state: started
        enabled: yes
    
    - name: Create web directory
      file:
        path: /var/www/lab
        state: directory
        owner: www-data
        group: www-data
        mode: '0755'
    
    - name: Deploy custom index page
      copy:
        content: |
          <!DOCTYPE html>
          <html>
          <head>
              <title>Ansible + Terraform Lab</title>
              <style>
                  body {
                      font-family: Arial, sans-serif;
                      text-align: center;
                      background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
                      color: white;
                      padding: 50px;
                  }
                  .container {
                      background: rgba(255,255,255,0.1);
                      padding: 40px;
                      border-radius: 10px;
                      backdrop-filter: blur(10px);
                  }
                  h1 { font-size: 3em; margin-bottom: 20px; }
                  p { font-size: 1.2em; }
              </style>
          </head>
          <body>
              <div class="container">
                  <h1>🚀 Success!</h1>
                  <p>Infrastructure provisioned with <strong>Terraform</strong></p>
                  <p>Configured with <strong>Ansible</strong></p>
                  <p>Deployed on <strong>AWS</strong></p>
                  <hr>
                  <p>Server: {{ ansible_hostname }}</p>
                  <p>IP: {{ ansible_default_ipv4.address }}</p>
              </div>
          </body>
          </html>
        dest: /var/www/lab/index.html
        owner: www-data
        group: www-data
        mode: '0644'
    
    - name: Configure Nginx site
      copy:
        content: |
          server {
              listen 80 default_server;
              listen [::]:80 default_server;
              root /var/www/lab;
              index index.html;
              server_name _;
              location / {
                  try_files $uri $uri/ =404;
              }
          }
        dest: /etc/nginx/sites-available/lab
    
    - name: Enable site
      file:
        src: /etc/nginx/sites-available/lab
        dest: /etc/nginx/sites-enabled/lab
        state: link
    
    - name: Remove default site
      file:
        path: /etc/nginx/sites-enabled/default
        state: absent
    
    - name: Test Nginx configuration
      command: nginx -t
      changed_when: false
    
    - name: Reload Nginx
      service:
        name: nginx
        state: reloaded
    
    - name: Display success message
      debug:
        msg: "Web server configured! Visit http://{{ ansible_default_ipv4.address }}"
```

- [ ] Web server playbook created

#### 5.6 Run the Playbook

```bash
cd ~/ansible-terraform-lab/ansible
ansible-playbook webserver-setup.yml
```

- [ ] Playbook executed successfully
- [ ] Nginx installed and configured

#### 5.7 Verify Web Server

Open a browser and visit: `http://<INSTANCE_PUBLIC_IP>`

- [ ] Web page loads successfully
- [ ] Custom page displayed

---

## Lab 6: Complete Integration Project

### Theory: End-to-End Automation

This lab combines everything you've learned to create a complete, automated deployment pipeline:
1. Terraform provisions infrastructure
2. Ansible configures and deploys applications
3. Everything is automated and repeatable

### Tasks

#### 6.1 Create Variables File

Create `terraform/variables.tf`:

```hcl
variable "aws_region" {
  description = "AWS region"
  type        = string
  default     = "us-east-1"
}

variable "instance_type" {
  description = "EC2 instance type"
  type        = string
  default     = "t2.micro"
}

variable "project_name" {
  description = "Project name for tagging"
  type        = string
  default     = "ansible-terraform-lab"
}
```

Update `terraform/main.tf` to use variables:

```hcl
provider "aws" {
  region = var.aws_region
}
```

- [ ] Variables file created
- [ ] Main.tf updated to use variables

#### 6.2 Create Ansible Role Structure

```bash
cd ~/ansible-terraform-lab/ansible
mkdir -p roles/webapp/{tasks,templates,handlers,files}
```

Create `roles/webapp/tasks/main.yml`:

```yaml
---
- name: Install required packages
  apt:
    name:
      - nginx
      - python3-pip
      - git
    state: present
    update_cache: yes

- name: Deploy application
  template:
    src: webapp.html.j2
    dest: /var/www/html/index.html
    owner: www-data
    group: www-data
    mode: '0644'
  notify: reload nginx

- name: Ensure Nginx is running
  service:
    name: nginx
    state: started
    enabled: yes
```

Create `roles/webapp/handlers/main.yml`:

```yaml
---
- name: reload nginx
  service:
    name: nginx
    state: reloaded
```

Create `roles/webapp/templates/webapp.html.j2`:

```html
<!DOCTYPE html>
<html>
<head>
    <title>{{ project_name }}</title>
    <style>
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            margin: 0;
            padding: 0;
            background: linear-gradient(135deg, #1e3c72 0%, #2a5298 100%);
            color: white;
        }
        .header {
            background: rgba(0,0,0,0.3);
            padding: 20px;
            text-align: center;
        }
        .container {
            max-width: 800px;
            margin: 50px auto;
            padding: 40px;
            background: rgba(255,255,255,0.1);
            border-radius: 15px;
            backdrop-filter: blur(10px);
            box-shadow: 0 8px 32px 0 rgba(31, 38, 135, 0.37);
        }
        h1 { font-size: 2.5em; margin: 0; }
        .info-grid {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
            gap: 20px;
            margin-top: 30px;
        }
        .info-card {
            background: rgba(255,255,255,0.1);
            padding: 20px;
            border-radius: 10px;
        }
        .info-card h3 { margin-top: 0; color: #4CAF50; }
    </style>
</head>
<body>
    <div class="header">
        <h1>🎯 {{ project_name | upper }}</h1>
        <p>Infrastructure as Code in Action</p>
    </div>
    <div class="container">
        <h2>✅ Deployment Successful</h2>
        <div class="info-grid">
            <div class="info-card">
                <h3>🏗️ Infrastructure</h3>
                <p>Provisioned with Terraform</p>
            </div>
            <div class="info-card">
                <h3>⚙️ Configuration</h3>
                <p>Managed by Ansible</p>
            </div>
            <div class="info-card">
                <h3>☁️ Cloud Provider</h3>
                <p>Amazon Web Services</p>
            </div>
            <div class="info-card">
                <h3>🖥️ Server Info</h3>
                <p>{{ ansible_hostname }}</p>
                <p>{{ ansible_distribution }} {{ ansible_distribution_version }}</p>
            </div>
        </div>
    </div>
</body>
</html>
```

- [ ] Role structure created
- [ ] Tasks, handlers, and templates created

#### 6.3 Create Master Playbook

Create `ansible/deploy.yml`:

```yaml
---
- name: Complete Application Deployment
  hosts: aws_ec2
  become: yes
  gather_facts: yes
  
  vars:
    project_name: "Ansible Terraform Lab"
  
  roles:
    - webapp
  
  post_tasks:
    - name: Display deployment information
      debug:
        msg:
          - "=========================================="
          - "Deployment completed successfully!"
          - "Web URL: http://{{ ansible_default_ipv4.address }}"
          - "SSH: ssh -i ~/.ssh/aws-lab-key ubuntu@{{ ansible_default_ipv4.address }}"
          - "=========================================="
```

- [ ] Master playbook created

#### 6.4 Create Deployment Script

Create `deploy.sh` in the project root:

```bash
#!/bin/bash

set -e

echo "================================"
echo "Starting Deployment Pipeline"
echo "================================"

# Step 1: Provision Infrastructure
echo -e "\n[1/3] Provisioning AWS Infrastructure with Terraform..."
cd terraform
terraform init -upgrade
terraform apply -auto-approve

# Wait for instance to be ready
echo "Waiting for instance to initialize..."
sleep 30

# Step 2: Configure with Ansible
echo -e "\n[2/3] Configuring infrastructure with Ansible..."
cd ../ansible

# Test connectivity
echo "Testing connectivity..."
ansible aws_ec2 -m ping

# Deploy application
echo "Deploying application..."
ansible-playbook deploy.yml

# Step 3: Get outputs
echo -e "\n[3/3] Deployment Complete!"
cd ../terraform
INSTANCE_IP=$(terraform output -raw instance_public_ip)

echo "================================"
echo "Access your application at:"
echo "http://$INSTANCE_IP"
echo "================================"
```

Make it executable:

```bash
chmod +x deploy.sh
```

- [ ] Deployment script created
- [ ] Script is executable

#### 6.5 Run Complete Deployment

```bash
cd ~/ansible-terraform-lab
./deploy.sh
```

- [ ] Deployment script executed successfully
- [ ] Infrastructure provisioned
- [ ] Application configured and deployed
- [ ] Web application accessible

#### 6.6 Create Destroy Script

Create `destroy.sh`:

```bash
#!/bin/bash

set -e

echo "================================"
echo "Destroying Infrastructure"
echo "================================"

cd terraform
terraform destroy -auto-approve

echo "Infrastructure destroyed successfully!"
```

Make it executable:

```bash
chmod +x destroy.sh
```

- [ ] Destroy script created
- [ ] Script is executable

---

## Lab 7: Advanced Topics

### Theory: Production-Ready Practices

For production environments, consider:
- **Remote State**: Store Terraform state in S3 with DynamoDB locking
- **Secrets Management**: Use AWS Secrets Manager or Ansible Vault
- **Monitoring**: CloudWatch, Prometheus, Grafana
- **High Availability**: Multi-AZ deployments, Auto Scaling Groups
- **CI/CD Integration**: Jenkins, GitLab CI, GitHub Actions

### Tasks

#### 7.1 Configure Terraform Remote State

Create `terraform/backend.tf`:

```hcl
terraform {
  backend "s3" {
    bucket         = "my-terraform-state-bucket"
    key            = "ansible-terraform-lab/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
    dynamodb_table = "terraform-state-lock"
  }
}
```

**Note**: You'll need to create the S3 bucket and DynamoDB table first.

- [ ] Remote state configuration reviewed
- [ ] Understand remote state benefits

#### 7.2 Use Ansible Vault for Secrets

Create encrypted variables:

```bash
cd ~/ansible-terraform-lab/ansible
ansible-vault create vars/secrets.yml
```

Enter a password and add:

```yaml
---
db_password: "super_secret_password"
api_key: "your_api_key_here"
```

Update playbook to use secrets:

```yaml
---
- name: Deploy with Secrets
  hosts: aws_ec2
  become: yes
  vars_files:
    - vars/secrets.yml
  
  tasks:
    - name: Use secret in task
      debug:
        msg: "DB Password is secured"
```

Run with:

```bash
ansible-playbook deploy.yml --ask-vault-pass
```

- [ ] Ansible Vault file created
- [ ] Understand secrets management

#### 7.3 Implement Auto Scaling (Theory)

**Concept**: Auto Scaling Groups (ASG) automatically adjust EC2 instance count based on demand.

Example Terraform configuration:

```hcl
resource "aws_launch_template" "web" {
  name_prefix   = "web-"
  image_id      = data.aws_ami.ubuntu.id
  instance_type = "t2.micro"
  
  user_data = base64encode(templatefile("userdata.sh", {
    ansible_playbook_url = "https://your-repo/playbook.yml"
  }))
}

resource "aws_autoscaling_group" "web" {
  desired_capacity    = 2
  max_size           = 5
  min_size           = 1
  target_group_arns  = [aws_lb_target_group.web.arn]
  vpc_zone_identifier = [aws_subnet.public_subnet.id]
  
  launch_template {
    id      = aws_launch_template.web.id
    version = "$Latest"
  }
}
```

- [ ] Auto Scaling concept reviewed
- [ ] Understand scaling strategies

#### 7.4 Add Application Load Balancer (Theory)

```hcl
resource "aws_lb" "web" {
  name               = "web-alb"
  internal           = false
  load_balancer_type = "application"
  security_groups    = [aws_security_group.alb_sg.id]
  subnets            = [aws_subnet.public_subnet_1.id, aws_subnet.public_subnet_2.id]
}

resource "aws_lb_target_group" "web" {
  name     = "web-tg"
  port     = 80
  protocol = "HTTP"
  vpc_id   = aws_vpc.lab_vpc.id
  
  health_check {
    path                = "/"
    healthy_threshold   = 2
    unhealthy_threshold = 10
  }
}

resource "aws_lb_listener" "web" {
  load_balancer_arn = aws_lb.web.arn
  port              = "80"
  protocol          = "HTTP"
  
  default_action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.web.arn
  }
}
```

- [ ] Load balancer concept reviewed
- [ ] Understand high availability patterns

#### 7.5 Ansible Dynamic Inventories from Terraform

Create `terraform/ansible-inventory.tf`:

```hcl
resource "local_file" "ansible_inventory" {
  content = templatefile("${path.module}/templates/inventory.tpl", {
    instances = aws_instance.web_server.*.public_ip
  })
  filename = "${path.module}/../inventory/generated_hosts.ini"
}
```

Create `terraform/templates/inventory.tpl`:

```ini
[web_servers]
%{ for ip in instances ~}
${ip}
%{ endfor ~}

[web_servers:vars]
ansible_user=ubuntu
ansible_ssh_private_key_file=~/.ssh/aws-lab-key
```

- [ ] Dynamic inventory generation reviewed
- [ ] Understand Terraform-Ansible integration patterns

#### 7.6 CI/CD Integration Example

Example GitHub Actions workflow (`.github/workflows/deploy.yml`):

```yaml
name: Deploy Infrastructure

on:
  push:
    branches: [ main ]
  workflow_dispatch:

jobs:
  terraform:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      
      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v1
      
      - name: Terraform Init
        run: terraform init
        working-directory: ./terraform
      
      - name: Terraform Apply
        run: terraform apply -auto-approve
        working-directory: ./terraform
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
  
  ansible:
    needs: terraform
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      
      - name: Setup Python
        uses: actions/setup-python@v2
        with:
          python-version: '3.9'
      
      - name: Install Ansible
        run: |
          pip install ansible boto3
          ansible-galaxy collection install amazon.aws
      
      - name: Run Ansible Playbook
        run: ansible-playbook deploy.yml
        working-directory: ./ansible
```

- [ ] CI/CD concept reviewed
- [ ] Understand automation pipeline

#### 7.7 Monitoring and Logging Setup

Create `ansible/roles/monitoring/tasks/main.yml`:

```yaml
---
- name: Install CloudWatch agent
  apt:
    deb: https://s3.amazonaws.com/amazoncloudwatch-agent/ubuntu/amd64/latest/amazon-cloudwatch-agent.deb

- name: Configure CloudWatch agent
  template:
    src: cloudwatch-config.json.j2
    dest: /opt/aws/amazon-cloudwatch-agent/etc/config.json
  notify: restart cloudwatch agent

- name: Start CloudWatch agent
  service:
    name: amazon-cloudwatch-agent
    state: started
    enabled: yes
```

- [ ] Monitoring concepts reviewed
- [ ] Understand observability practices

---

## Cleanup

### Final Cleanup Tasks

```bash
cd ~/ansible-terraform-lab
./destroy.sh
```

Verify all resources are destroyed:

```bash
aws ec2 describe-instances --filters "Name=tag:Name,Values=ansible-terraform-lab-*" --query "Reservations[].Instances[].State.Name"
```

- [ ] All AWS resources destroyed
- [ ] Terraform state cleaned
- [ ] No remaining charges

---

## Summary and Next Steps

### What You've Learned

✅ **Terraform Fundamentals**
- Infrastructure as Code basics
- Provider configuration
- Resource management
- State management
- Outputs and variables

✅ **Ansible Fundamentals**
- Playbook creation
- Inventory management
- Modules and tasks
- Roles and organization
- Idempotency

✅ **AWS Services**
- VPC and networking
- EC2 instances
- Security groups
- Key pairs
- S3 buckets

✅ **Integration Patterns**
- Terraform + Ansible workflow
- Dynamic inventories
- Automated deployments
- Infrastructure lifecycle management

### Next Steps

1. **Explore More AWS Services**
   - RDS databases
   - ElastiCache
   - Lambda functions
   - ECS/EKS containers

2. **Advanced Terraform**
   - Modules
   - Workspaces
   - Complex dependencies
   - Import existing infrastructure

3. **Advanced Ansible**
   - Custom modules
   - Ansible Tower/AWX
   - Complex roles
   - Testing with Molecule

4. **Production Practices**
   - Multi-environment deployments
   - Blue-green deployments
   - Disaster recovery
   - Cost optimization

### Resources

- [Terraform Documentation](https://www.terraform.io/docs)
- [Ansible Documentation](https://docs.ansible.com)
- [AWS Documentation](https://docs.aws.amazon.com)
- [HashiCorp Learn](https://learn.hashicorp.com)
- [Ansible Galaxy](https://galaxy.ansible.com)

---

**Congratulations!**

You've completed the Ansiblelaboratory. 
Keep practicing and building more complex infrastructures.

Kirk Patrick