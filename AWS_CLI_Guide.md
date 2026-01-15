# AWS CLI Quick Reference

## Basic Command Pattern

```powershell
aws <service> <command> [options]
```

## Most Common Commands

### Check Installation
```powershell
# Test if AWS CLI is installed
aws --version
```

### Configure AWS CLI
```powershell
# Configure credentials (interactive)
aws configure
```

You'll need:
- Access Key ID
- Secret Access Key
- Default region (e.g., `us-east-1`)
- Output format (e.g., `json`)

### Basic Commands
```powershell
# Get your AWS account identity
aws sts get-caller-identity

# List S3 buckets
aws s3 ls

# List EC2 instances
aws ec2 describe-instances

# Help for any service
aws ec2 help
aws s3 help
```

## Create and Start an EC2 Instance

### 1. Launch a New Instance
```powershell
aws ec2 run-instances `
    --image-id ami-xxxxxxxxx `
    --instance-type t2.micro `
    --key-name your-key-pair `
    --security-group-ids sg-xxxxxxxxx `
    --subnet-id subnet-xxxxxxxxx `
    --count 1
```

### 2. Find Available AMIs
```powershell
# List Amazon Linux 2 AMIs
aws ec2 describe-images `
    --owners amazon `
    --filters "Name=name,Values=amzn2-ami-hvm-*" `
    --query 'Images[*].[ImageId,Name,CreationDate]' `
    --output table
```

### 3. Start an Existing Stopped Instance
```powershell
aws ec2 start-instances --instance-ids i-xxxxxxxxx
```

### 4. To check instance status:
```powershell
aws ec2 describe-instances --instance-ids i-xxxxxxxxx
```

### 5. To get your instance ID after creation:
```powershell
aws ec2 describe-instances `
    --filters "Name=instance-state-name,Values=running" `
    --query 'Reservations[*].Instances[*].[InstanceId,State.Name,PublicIpAddress]' `
    --output table
```

## Prerequisites

- AWS CLI installed and configured (`aws configure`)
- Valid key pair: `aws ec2 create-key-pair --key-name my-key`
- Security group allowing SSH: `aws ec2 create-security-group`

## Installation

If AWS CLI is not installed, download from: https://aws.amazon.com/cli/

## Notes

- The `run-instances` command automatically starts the instance after creation
- You only need `start-instances` for previously stopped instances
