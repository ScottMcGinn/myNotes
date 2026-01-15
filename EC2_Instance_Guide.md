# Creating and Starting EC2 Instances with AWS CLI

## Prerequisites

Before creating an EC2 instance, ensure you have:

1. **AWS CLI installed and configured**
   ```powershell
   aws configure
   ```

2. **Key Pair** (for SSH access)
   ```powershell
   # Create a new key pair
   aws ec2 create-key-pair --key-name my-key-pair --query 'KeyMaterial' --output text > my-key-pair.pem
   
   # List existing key pairs
   aws ec2 describe-key-pairs
   ```

3. **Security Group** (for network access)
   ```powershell
   # Create a security group
   aws ec2 create-security-group --group-name my-sg --description "My security group"
   
   # Allow SSH access (port 22)
   aws ec2 authorize-security-group-ingress --group-name my-sg --protocol tcp --port 22 --cidr 0.0.0.0/0
   
   # List security groups
   aws ec2 describe-security-groups
   ```

## Step 1: Find an Amazon Machine Image (AMI)

```powershell
# Find Amazon Linux 2 AMIs
aws ec2 describe-images `
    --owners amazon `
    --filters "Name=name,Values=amzn2-ami-hvm-*" "Name=state,Values=available" `
    --query 'Images | sort_by(@, &CreationDate) | [-1].[ImageId,Name,Description]' `
    --output table

# Find Ubuntu AMIs
aws ec2 describe-images `
    --owners 099720109477 `
    --filters "Name=name,Values=ubuntu/images/hvm-ssd/ubuntu-*-amd64-server-*" "Name=state,Values=available" `
    --query 'Images | sort_by(@, &CreationDate) | [-1].[ImageId,Name,Description]' `
    --output table

# Find Windows Server AMIs
aws ec2 describe-images `
    --owners amazon `
    --filters "Name=name,Values=Windows_Server-*" "Name=state,Values=available" `
    --query 'Images | sort_by(@, &CreationDate) | [-5:].[ImageId,Name]' `
    --output table
```

## Step 2: Launch a New EC2 Instance

### Basic Launch (Simplest)
```powershell
aws ec2 run-instances `
    --image-id ami-xxxxxxxxx `
    --instance-type t2.micro `
    --key-name my-key-pair `
    --count 1
```

### Complete Launch with All Options
```powershell
aws ec2 run-instances `
    --image-id ami-xxxxxxxxx `
    --instance-type t2.micro `
    --key-name my-key-pair `
    --security-group-ids sg-xxxxxxxxx `
    --subnet-id subnet-xxxxxxxxx `
    --associate-public-ip-address `
    --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=MyInstance}]' `
    --count 1
```

### Launch with User Data Script
```powershell
aws ec2 run-instances `
    --image-id ami-xxxxxxxxx `
    --instance-type t2.micro `
    --key-name my-key-pair `
    --user-data file://startup-script.sh `
    --count 1
```

## Step 3: Get Instance Information

### Get Instance ID from Launch Output
The `run-instances` command returns JSON with instance details. Save the Instance ID.

### List All Running Instances
```powershell
aws ec2 describe-instances `
    --filters "Name=instance-state-name,Values=running" `
    --query 'Reservations[*].Instances[*].[InstanceId,InstanceType,State.Name,PublicIpAddress,Tags[?Key==`Name`].Value|[0]]' `
    --output table
```

### Get Specific Instance Details
```powershell
aws ec2 describe-instances --instance-ids i-xxxxxxxxx
```

### Get Just the Public IP
```powershell
aws ec2 describe-instances `
    --instance-ids i-xxxxxxxxx `
    --query 'Reservations[0].Instances[0].PublicIpAddress' `
    --output text
```

## Step 4: Manage Instance State

### Start a Stopped Instance
```powershell
aws ec2 start-instances --instance-ids i-xxxxxxxxx
```

### Stop a Running Instance
```powershell
aws ec2 stop-instances --instance-ids i-xxxxxxxxx
```

### Reboot an Instance
```powershell
aws ec2 reboot-instances --instance-ids i-xxxxxxxxx
```

### Terminate (Delete) an Instance
```powershell
aws ec2 terminate-instances --instance-ids i-xxxxxxxxx
```

## Step 5: Monitor Instance Status

### Check Instance Status
```powershell
aws ec2 describe-instance-status --instance-ids i-xxxxxxxxx
```

### Wait for Instance to be Running
```powershell
aws ec2 wait instance-running --instance-ids i-xxxxxxxxx
Write-Host "Instance is now running"
```

### Wait for Instance to be Stopped
```powershell
aws ec2 wait instance-stopped --instance-ids i-xxxxxxxxx
Write-Host "Instance is now stopped"
```

## Common Instance Types

- `t2.micro` - 1 vCPU, 1 GB RAM (Free tier eligible)
- `t2.small` - 1 vCPU, 2 GB RAM
- `t2.medium` - 2 vCPU, 4 GB RAM
- `t3.micro` - 2 vCPU, 1 GB RAM
- `t3.small` - 2 vCPU, 2 GB RAM

## Complete Example Workflow

```powershell
# 1. Find latest Amazon Linux 2 AMI ID
$AMI_ID = aws ec2 describe-images `
    --owners amazon `
    --filters "Name=name,Values=amzn2-ami-hvm-*" "Name=state,Values=available" `
    --query 'Images | sort_by(@, &CreationDate) | [-1].ImageId' `
    --output text

# 2. Launch instance
$INSTANCE_ID = aws ec2 run-instances `
    --image-id $AMI_ID `
    --instance-type t2.micro `
    --key-name my-key-pair `
    --query 'Instances[0].InstanceId' `
    --output text

Write-Host "Launched instance: $INSTANCE_ID"

# 3. Wait for instance to be running
aws ec2 wait instance-running --instance-ids $INSTANCE_ID

# 4. Get public IP
$PUBLIC_IP = aws ec2 describe-instances `
    --instance-ids $INSTANCE_ID `
    --query 'Reservations[0].Instances[0].PublicIpAddress' `
    --output text

Write-Host "Instance is running at: $PUBLIC_IP"
Write-Host "Connect with: ssh -i my-key-pair.pem ec2-user@$PUBLIC_IP"
```

## Troubleshooting

### Instance won't start
```powershell
# Check instance status checks
aws ec2 describe-instance-status --instance-ids i-xxxxxxxxx

# Check system logs
aws ec2 get-console-output --instance-ids i-xxxxxxxxx
```

### Can't connect via SSH
- Verify security group allows port 22
- Check that you're using the correct key pair
- Ensure instance has a public IP address

### Find default VPC and Subnet
```powershell
# Get default VPC
aws ec2 describe-vpcs --filters "Name=isDefault,Values=true"

# Get subnets in default VPC
aws ec2 describe-subnets --filters "Name=default-for-az,Values=true"
```

## Important Notes

- `run-instances` automatically starts the instance - no need to call `start-instances`
- Instances continue running (and incurring charges) until you stop or terminate them
- Use `stop-instances` to pause (data preserved, no compute charges)
- Use `terminate-instances` to delete permanently (cannot be undone)
- Always use `t2.micro` for free tier eligibility (first 12 months, 750 hours/month)
