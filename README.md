# 🚀 Custom Golden AMI Creation with Packer & GitHub Actions

[![Build Status](https://github.com/maheshnfunde2000/aws-golden-ami-automation/workflows/Build%20AMI/badge.svg)](https://github.com/maheshnfunde2000/aws-golden-ami-automation/actions)
[![AWS](https://img.shields.io/badge/AWS-EC2-orange.svg)](https://aws.amazon.com/ec2/)
[![Packer](https://img.shields.io/badge/Packer-v1.9+-blue.svg)](https://www.packer.io/)

## 📋 Table of Contents
- [Overview](#overview)
- [Architecture](#architecture)
- [Prerequisites](#prerequisites)
- [Project Structure](#project-structure)
- [Manual Golden AMI Creation](#manual-golden-ami-creation)
- [Automated CI/CD Pipeline Setup](#automated-cicd-pipeline-setup)
- [Configuration](#configuration)
- [Usage](#usage)
- [Customization](#customization)
- [Troubleshooting](#troubleshooting)

## 🎯 Overview

This project demonstrates how to create **Golden AMIs** (Amazon Machine Images) using HashiCorp Packer with automated CI/CD pipelines via GitHub Actions. Golden AMIs are pre-configured, hardened, and optimized base images that serve as standardized templates for launching EC2 instances across your infrastructure.

### 🌟 Key Features
- **Automated AMI Creation** with Packer
- **CI/CD Integration** using GitHub Actions
- **Multi-Environment Support** (Dev, Staging, Production)
- **Security Hardening** with custom configurations
- **Version Control** for infrastructure as code
- **Cost Optimization** through reusable images

### 🎬 Demo & Tutorial
- 📹 **Video Tutorial**: [Watch on YouTube](your-youtube-link)
- 🖼️ **Screenshots**: Available in `/docs/screenshots/`

## 🏗️ Architecture

```mermaid
graph TB
    A[GitHub Repository] --> B[GitHub Actions Workflow]
    B --> C[Packer Build Process]
    C --> D[AWS EC2 Instance]
    D --> E[Install & Configure Software]
    E --> F[Create Golden AMI]
    F --> G[AMI Available in AWS]
    G --> H[Launch Production Instances]
    
    subgraph "Packer Build Process"
        C --> I[Source Block - Base AMI]
        I --> J[Provisioners - Shell/Ansible]
        J --> K[Post-processors]
    end
    
    subgraph "AWS Resources"
        D --> L[Security Groups]
        D --> M[IAM Roles]
        G --> N[AMI Catalog]
    end
```

### 🔄 Workflow Architecture

```mermaid
sequenceDiagram
    participant Dev as Developer
    participant GH as GitHub
    participant GA as GitHub Actions
    participant P as Packer
    participant AWS as AWS EC2
    
    Dev->>GH: Push code changes
    GH->>GA: Trigger workflow
    GA->>P: Initialize Packer
    P->>AWS: Launch temporary instance
    P->>AWS: Run provisioning scripts
    P->>AWS: Create AMI snapshot
    AWS->>P: Return AMI ID
    P->>GA: Build complete
    GA->>GH: Update build status
```

## 📋 Prerequisites

### 🔧 Required Tools
- **AWS Account** with appropriate permissions
- **GitHub Account** for repository and CI/CD
- **Packer** (v1.9.0 or later) - for local testing
- **AWS CLI** (optional) - for manual operations

### 🔐 AWS Permissions Required
Your IAM user/role needs the following permissions:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ec2:AttachVolume",
                "ec2:AuthorizeSecurityGroupIngress",
                "ec2:CopyImage",
                "ec2:CreateImage",
                "ec2:CreateKeypair",
                "ec2:CreateSecurityGroup",
                "ec2:CreateSnapshot",
                "ec2:CreateTags",
                "ec2:CreateVolume",
                "ec2:DeleteKeyPair",
                "ec2:DeleteSecurityGroup",
                "ec2:DeleteSnapshot",
                "ec2:DeleteVolume",
                "ec2:DeregisterImage",
                "ec2:DescribeImageAttribute",
                "ec2:DescribeImages",
                "ec2:DescribeInstances",
                "ec2:DescribeInstanceStatus",
                "ec2:DescribeRegions",
                "ec2:DescribeSecurityGroups",
                "ec2:DescribeSnapshots",
                "ec2:DescribeVolumes",
                "ec2:DetachVolume",
                "ec2:GetPasswordData",
                "ec2:ModifyImageAttribute",
                "ec2:ModifyInstanceAttribute",
                "ec2:ModifySnapshotAttribute",
                "ec2:RegisterImage",
                "ec2:RunInstances",
                "ec2:StopInstances",
                "ec2:TerminateInstances"
            ],
            "Resource": "*"
        }
    ]
}
```

## 📁 Project Structure

```
aws-golden-ami-automation/
├── .github/
│   └── workflows/
│       └── build-ami.yml          # GitHub Actions workflow
├── packer/
│   ├── templates/
│   │   ├── ubuntu-golden.pkr.hcl  # Main Packer template
│   │   └── variables.pkr.hcl      # Variable definitions
│   └── scripts/
│       ├── install-base.sh        # Base software installation
│       ├── install-docker.sh      # Docker installation
│       ├── install-monitoring.sh  # Monitoring tools
│       └── cleanup.sh             # System cleanup
├── docs/
│   ├── screenshots/               # Project screenshots
│   └── architecture.md            # Detailed architecture docs
├── examples/
│   └── terraform/                 # Example Terraform configs
├── README.md                      # This file
├── LICENSE                        # Project license
└── .gitignore                     # Git ignore rules
```

## 🔧 Manual Golden AMI Creation

Follow these steps to create your Golden AMI manually using Packer on your local machine.

### Step 1: Local Environment Setup

```bash
# Clone the repository
git clone https://github.com/maheshnfunde2000/aws-golden-ami-automation.git
cd aws-golden-ami-automation

# Install Packer (if not already installed)
# For macOS
brew install packer

# For Ubuntu/Debian
curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo apt-key add -
sudo apt-add-repository "deb [arch=amd64] https://apt.releases.hashicorp.com $(lsb_release -cs) main"
sudo apt-get update && sudo apt-get install packer

# For Windows (using Chocolatey)
choco install packer
```

### Step 2: AWS CLI Configuration

```bash
# Install AWS CLI if not already installed
pip install awscli

# Configure AWS credentials
aws configure
# Enter your AWS Access Key ID
# Enter your AWS Secret Access Key
# Enter your default region (e.g., us-east-1)
# Enter default output format (json)

# Verify configuration
aws sts get-caller-identity
```

### Step 3: Customize Packer Configuration

Edit the variables in `packer/templates/variables.pkr.hcl`:

```hcl
variable "aws_region" {
  description = "AWS region"
  type        = string
  default     = "us-east-1"  # Change to your preferred region
}

variable "instance_type" {
  description = "EC2 instance type for building"
  type        = string
  default     = "t3.micro"   # Adjust based on your needs
}

variable "ami_name_prefix" {
  description = "Prefix for AMI name"
  type        = string
  default     = "golden-ubuntu"  # Customize your AMI name
}
```

### Step 4: Validate and Build AMI

```bash
# Navigate to packer templates directory
cd packer/templates

# Validate the Packer configuration
packer validate ubuntu-golden.pkr.hcl

# Initialize Packer (download required plugins)
packer init .

# Build the AMI
packer build ubuntu-golden.pkr.hcl

# Monitor the build process - it will show:
# - Instance launch
# - Provisioner execution
# - AMI creation
# - Cleanup process
```

### Step 5: Verify AMI Creation

```bash
# List your custom AMIs
aws ec2 describe-images --owners self --query 'Images[*].[ImageId,Name,CreationDate]' --output table

# Get the latest AMI ID
AMI_ID=$(aws ec2 describe-images --owners self --query 'Images | sort_by(@, &CreationDate) | [-1].ImageId' --output text)
echo "Latest AMI ID: $AMI_ID"
```

### Step 6: Test the AMI

```bash
# Launch a test instance using your Golden AMI
aws ec2 run-instances \
    --image-id $AMI_ID \
    --instance-type t3.micro \
    --key-name your-key-pair-name \
    --security-group-ids your-security-group-id \
    --subnet-id your-subnet-id \
    --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=Golden-AMI-Test}]'
```

---

## 🚀 Automated CI/CD Pipeline Setup

Set up automated AMI creation using GitHub Actions for continuous integration and deployment.

### Step 1: Fork Repository

```bash
# Fork the repository on GitHub or clone it
git clone https://github.com/maheshnfunde2000/aws-golden-ami-automation.git
cd aws-golden-ami-automation
```

### Step 2: AWS IAM Setup for CI/CD

Create a dedicated IAM user for GitHub Actions:

```bash
# Create IAM user via AWS CLI
aws iam create-user --user-name packer-github-actions

# Attach the required policy (created from the JSON in Prerequisites section)
aws iam attach-user-policy --user-name packer-github-actions --policy-arn arn:aws:iam::YOUR-ACCOUNT:policy/PackerBuildPolicy

# Create access keys
aws iam create-access-key --user-name packer-github-actions
```

### Step 3: Configure GitHub Repository Secrets

Navigate to your GitHub repository → **Settings** → **Secrets and Variables** → **Actions**

Click **"New repository secret"** and add the following:

| Secret Name | Description | Example Value |
|-------------|-------------|---------------|
| `AWS_ACCESS_KEY_ID` | AWS Access Key ID | `AKIAIOSFODNN7EXAMPLE` |
| `AWS_SECRET_ACCESS_KEY` | AWS Secret Access Key | `wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY` |
| `AWS_DEFAULT_REGION` | Default AWS Region | `us-east-1` |

### Step 4: Customize GitHub Actions Workflow

Review and modify `.github/workflows/build-ami.yml`:

```yaml
name: Build Golden AMI

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]
  workflow_dispatch:  # Manual trigger
  schedule:
    - cron: '0 2 * * 0'  # Weekly build every Sunday at 2 AM

env:
  AWS_DEFAULT_REGION: ${{ secrets.AWS_DEFAULT_REGION }}
  PACKER_VERSION: "1.9.4"

jobs:
  build-ami:
    runs-on: ubuntu-latest
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v3
    
    - name: Setup Packer
      uses: hashicorp/setup-packer@main
      with:
        version: ${{ env.PACKER_VERSION }}
    
    - name: Configure AWS credentials
      uses: aws-actions/configure-aws-credentials@v2
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-region: ${{ secrets.AWS_DEFAULT_REGION }}
    
    - name: Validate Packer template
      run: |
        cd packer/templates
        packer validate ubuntu-golden.pkr.hcl
    
    - name: Build AMI
      run: |
        cd packer/templates
        packer build ubuntu-golden.pkr.hcl
```

### Step 5: Environment-Specific Configurations

Create different configurations for multiple environments:

```bash
# Create environment-specific variable files
mkdir -p packer/environments

# Development environment
cat > packer/environments/dev.pkrvars.hcl << EOF
environment = "dev"
instance_type = "t3.nano"
ami_name_prefix = "golden-ubuntu-dev"
EOF

# Production environment
cat > packer/environments/prod.pkrvars.hcl << EOF
environment = "prod"
instance_type = "t3.small"
ami_name_prefix = "golden-ubuntu-prod"
EOF
```

### Step 6: Multi-Environment Workflow

Update your GitHub Actions workflow to support multiple environments:

```yaml
strategy:
  matrix:
    environment: [dev, staging, prod]

steps:
  - name: Build AMI for ${{ matrix.environment }}
    run: |
      cd packer/templates
      packer build -var-file="../environments/${{ matrix.environment }}.pkrvars.hcl" ubuntu-golden.pkr.hcl
```

### Step 7: Trigger Automated Build

```bash
# Make changes to your configuration
git add .
git commit -m "Setup automated AMI builds"
git push origin main

# Or trigger manually from GitHub Actions tab
# Go to Actions → Build Golden AMI → Run workflow
```

### Step 8: Monitor Build Progress

1. **GitHub Actions Tab**: Monitor real-time build progress
2. **AWS Console**: Check EC2 → AMIs for created images
3. **Build Artifacts**: View logs and build outputs

### Step 9: Automated Notifications (Optional)

Add Slack/Email notifications to your workflow:

```yaml
- name: Notify on Success
  if: success()
  uses: 8398a7/action-slack@v3
  with:
    status: success
    text: "✅ Golden AMI build completed successfully!"
  env:
    SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}

- name: Notify on Failure
  if: failure()
  uses: 8398a7/action-slack@v3
  with:
    status: failure
    text: "❌ Golden AMI build failed!"
  env:
    SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
```

## ⚙️ Configuration

### Packer Template Structure

The main Packer template (`ubuntu-golden.pkr.hcl`) includes:

```hcl
# Source block - defines the base image and build settings
source "amazon-ebs" "ubuntu" {
  ami_name      = "${var.ami_name_prefix}-{{timestamp}}"
  instance_type = var.instance_type
  region        = var.aws_region
  
  source_ami_filter {
    filters = {
      name                = var.source_ami_filter
      root-device-type    = "ebs"
      virtualization-type = "hvm"
    }
    most_recent = true
    owners      = ["099720109477"] # Canonical
  }
  
  ssh_username = "ubuntu"
  
  tags = {
    Name        = "${var.ami_name_prefix}-{{timestamp}}"
    Environment = "production"
    BuildDate   = "{{timestamp}}"
    BuildBy     = "packer"
  }
}

# Build block - defines what to do with the source
build {
  sources = ["source.amazon-ebs.ubuntu"]
  
  # Update system packages
  provisioner "shell" {
    script = "./scripts/install-base.sh"
  }
  
  # Install Docker
  provisioner "shell" {
    script = "./scripts/install-docker.sh"
  }
  
  # Install monitoring tools
  provisioner "shell" {
    script = "./scripts/install-monitoring.sh"
  }
  
  # Cleanup
  provisioner "shell" {
    script = "./scripts/cleanup.sh"
  }
}
```

### Provisioning Scripts

Each script serves a specific purpose:

#### `install-base.sh` - System Updates & Base Tools
```bash
#!/bin/bash
set -e

# Update system
sudo apt-get update -y
sudo apt-get upgrade -y

# Install essential tools
sudo apt-get install -y \
    curl \
    wget \
    git \
    vim \
    htop \
    unzip \
    jq \
    awscli

# Configure timezone
sudo timedatectl set-timezone UTC
```

#### `install-docker.sh` - Docker Installation
```bash
#!/bin/bash
set -e

# Install Docker
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Add ubuntu user to docker group
sudo usermod -aG docker ubuntu

# Enable Docker service
sudo systemctl enable docker
sudo systemctl start docker

# Install Docker Compose
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

## 🎮 Usage

### Manual Build (Local)

```bash
# Navigate to packer directory
cd packer/templates

# Validate configuration
packer validate ubuntu-golden.pkr.hcl

# Build AMI
packer build ubuntu-golden.pkr.hcl
```

### Automated Build (GitHub Actions)

The workflow automatically triggers on:
- Push to `main` branch
- Manual workflow dispatch
- Scheduled builds (weekly)

### Using the Generated AMI

```bash
# List your custom AMIs
aws ec2 describe-images --owners self --query 'Images[*].[ImageId,Name,CreationDate]' --output table

# Launch instance with your Golden AMI
aws ec2 run-instances \
    --image-id ami-xxxxxxxxx \
    --instance-type t3.micro \
    --key-name your-key-pair \
    --security-group-ids sg-xxxxxxxxx \
    --subnet-id subnet-xxxxxxxxx
```

### Terraform Example

```hcl
data "aws_ami" "golden_ubuntu" {
  most_recent = true
  owners      = ["self"]
  
  filter {
    name   = "name"
    values = ["golden-ubuntu-*"]
  }
}

resource "aws_instance" "web_server" {
  ami           = data.aws_ami.golden_ubuntu.id
  instance_type = "t3.micro"
  
  tags = {
    Name = "Web Server from Golden AMI"
  }
}
```

## 🔧 Customization

### Adding New Software

1. Create a new script in `packer/scripts/`:
```bash
# packer/scripts/install-nodejs.sh
#!/bin/bash
set -e

# Install Node.js
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt-get install -y nodejs

# Install global packages
sudo npm install -g pm2
```

2. Add to your Packer template:
```hcl
provisioner "shell" {
  script = "./scripts/install-nodejs.sh"
}
```

### Environment-Specific Builds

Create different templates for different environments:

```hcl
# variables.pkr.hcl
variable "environment" {
  description = "Environment name"
  type        = string
  default     = "dev"
}

# In your template
ami_name = "${var.ami_name_prefix}-${var.environment}-{{timestamp}}"
```

### Security Hardening

Add security configurations:

```bash
# packer/scripts/security-hardening.sh
#!/bin/bash
set -e

# Disable root login
sudo passwd -l root

# Configure SSH
sudo sed -i 's/#PermitRootLogin yes/PermitRootLogin no/' /etc/ssh/sshd_config
sudo sed -i 's/#PasswordAuthentication yes/PasswordAuthentication no/' /etc/ssh/sshd_config

# Install and configure fail2ban
sudo apt-get install -y fail2ban
sudo systemctl enable fail2ban

# Configure automatic updates
sudo apt-get install -y unattended-upgrades
sudo dpkg-reconfigure -plow unattended-upgrades
```

## 🐛 Troubleshooting

### Common Issues

#### 1. Build Fails with Permission Errors
```bash
Error: UnauthorizedOperation: You are not authorized to perform this operation
```
**Solution**: Verify AWS credentials and IAM permissions

#### 2. Instance Launch Timeout
```bash
Error: Timeout waiting for instance to become ready
```
**Solution**: Check security group settings and subnet configuration

#### 3. SSH Connection Failed
```bash
Error: SSH handshake failed
```
**Solution**: Ensure the security group allows SSH (port 22) from Packer's IP

### Debug Mode

Enable debug logging:

```bash
# Set environment variable
export PACKER_LOG=1
export PACKER_LOG_PATH="./packer.log"

# Run build
packer build ubuntu-golden.pkr.hcl
```

### Validation Commands

```bash
# Validate Packer template
packer validate ubuntu-golden.pkr.hcl

# Format Packer files
packer fmt ubuntu-golden.pkr.hcl

# Inspect template
packer inspect ubuntu-golden.pkr.hcl
```

## 📊 Monitoring & Costs

### Build Metrics
- **Average Build Time**: 10-15 minutes
- **Cost per Build**: ~$0.02 (t3.micro for 15 minutes)
- **AMI Storage Cost**: ~$0.045/month per GB

### Cost Optimization Tips
1. Use smaller instance types for building (t3.nano, t3.micro)
2. Delete old AMIs and snapshots regularly
3. Use spot instances for building (not recommended for production)
4. Optimize provisioning scripts for faster execution

## 🤝 Contributing

We welcome contributions! Please follow these steps:

1. **Fork** the repository
2. **Create** a feature branch (`git checkout -b feature/amazing-feature`)
3. **Commit** your changes (`git commit -m 'Add amazing feature'`)
4. **Push** to the branch (`git push origin feature/amazing-feature`)
5. **Open** a Pull Request



### Best practices
- Follow shell script best practices
- Test changes locally before submitting PR
- Update documentation for new features
- Add appropriate tags and comments


## 🙏 Acknowledgments

- [HashiCorp Packer](https://www.packer.io/) for the amazing tool
- [GitHub Actions](https://github.com/features/actions) for CI/CD platform
- [AWS](https://aws.amazon.com/) for cloud infrastructure
- Community contributors and feedback



🔗 **Connect with me**: [LinkedIn](https://linkedin.com/in/mahesh-funde) | [GitHub](https://github.com/maheshnfunde2000)
