# Terraform Execution Steps

This guide provides the exact step-by-step instructions for executing Terraform commands following the poc-idp repository workflow.

## Prerequisites

- Repository has been cloned and merged into current directory
- AWS CLI is configured with appropriate permissions
- S3 bucket exists for Terraform state storage

## Step-by-Step Execution Workflow

### Step 1: Create tfbackend File

**Ask Kiro to:**
> "Create tfbackend file for my project [client-name]-[aws-account-id]-[region]"

**File Location:** `tfbackend/[client-name]-[aws-account-id]-[region].tfbackend`

**File Content:**
```hcl
bucket = "your-terraform-state-bucket"
key    = "path/to/your/terraform.tfstate"
region = "your-aws-region"
```

**Important:**
- Use existing `*.tfbackend` files as reference
- Do NOT specify any credentials in the file
- Reference: https://developer.hashicorp.com/terraform/language/backend/s3

### Step 2: Create S3 Bucket

**Ask Kiro to:**
> "Create S3 bucket with proper permissions for Terraform state"

**Requirements:**
- S3 bucket with versioning enabled
- Proper IAM permissions for user executing Terraform
- Encryption enabled for state files

### Step 3: Create tfvars File

**Ask Kiro to:**
> "Create tfvars file [client-name]-[aws-account-id]-[region] using [size] setup"

**File Location:** `tfvars/[client-name]-[aws-account-id]-[region].tfvars`

**Template Selection:**
- `tfvars/example-small-setup.tfvars` - For Small setup
- `tfvars/example-medium-setup.tfvars` - For Medium setup
- `tfvars/example-large-setup.tfvars` - For Large setup

**Mandatory Variables to Modify:**
- `environment` - Environment name (dev, staging, prod)
- `account_id` - Your AWS account ID
- `region` - Your preferred AWS region
- `application` - Your application name

**Additional Configuration:**
- Check `README.md` for complete configuration options
- Replace all `CHANGE_ME_PLEASE` values
- Customize based on your specific requirements

### Step 4: Configure AWS Access

**Ask Kiro to:**
> "Verify AWS configuration and account access"

**Commands to Execute:**
```bash
# Verify AWS account
aws sts get-caller-identity
```

**Requirements:**
- Configure AWS access via environment variables or AWS CLI
- Ensure user has permissions for ECS, VPC, RDS, S3, IAM, and other AWS services
- Verify you're working with the correct AWS account

### Step 5: Initialize Terraform

**Ask Kiro to:**
> "Initialize Terraform with backend configuration"

**Command to Execute:**
```bash
terraform init -backend-config=tfbackends/[client-name]-[aws-account-id]-[region].tfbackend -reconfigure
```

**What This Does:**
- Initializes Terraform with your specific backend configuration
- Downloads required providers and modules
- Configures remote state storage in S3
- Uses `-reconfigure` flag to ensure clean initialization

### Step 6: Review Terraform Plan

**Ask Kiro to:**
> "Show me the Terraform plan for my infrastructure"

**Command to Execute:**
```bash
terraform plan -var-file=tfvars/[client-name]-[aws-account-id]-[region].tfvars
```

**What This Shows:**
- All resources that will be created
- Estimated costs and resource counts
- Configuration issues or conflicts
- Resource dependencies and creation order

**If Modifications Needed:**
1. Update your tfvars file based on plan review
2. Repeat the plan step to verify changes
3. Continue iterating until plan looks correct

### Step 7: Deploy Infrastructure

**Ask Kiro to:**
> "Deploy my ECS infrastructure"

**Command to Execute:**
```bash
terraform apply -var-file=tfvars/[client-name]-[aws-account-id]-[region].tfvars
```

**What This Does:**
- Executes the Terraform plan with confirmation
- Creates all AWS resources as defined in configuration
- Monitors deployment progress with real-time status
- Displays final outputs including endpoints and resource details

### Step 8: Verify Deployment

**Post-Deployment Validation:**
- ECS cluster is running and healthy
- Application Load Balancer health checks pass
- Database connectivity (if RDS/DocumentDB configured)
- All endpoints and URLs are accessible
- Security groups and networking configured correctly

## File Naming Conventions

**Strict Naming Requirements:**
- tfbackend file: `[client-name]-[aws-account-id]-[region].tfbackend`
- tfvars file: `[client-name]-[aws-account-id]-[region].tfvars`
- Use existing folder structure: `tfbackend/` and `tfvars/`

**Example:**
- Client: "acme"
- Account ID: "123456789012"
- Region: "us-east-1"
- Files: `acme-123456789012-us-east-1.tfbackend` and `acme-123456789012-us-east-1.tfvars`

## Command Reference

**Complete Command Sequence:**
```bash
# 1. Verify AWS account
aws sts get-caller-identity

# 2. Initialize Terraform
terraform init -backend-config=tfbackends/[client-name]-[account-id]-[region].tfbackend -reconfigure

# 3. Review plan
terraform plan -var-file=tfvars/[client-name]-[account-id]-[region].tfvars

# 4. Deploy infrastructure
terraform apply -var-file=tfvars/[client-name]-[account-id]-[region].tfvars
```

## Troubleshooting Common Issues

### Backend Configuration Issues
- Verify S3 bucket exists and is accessible
- Check IAM permissions for S3 and DynamoDB
- Ensure bucket versioning is enabled

### Variable Configuration Issues
- Verify all mandatory variables are set
- Check for typos in variable names
- Ensure account_id and region match your AWS setup

### Permission Issues
- Verify AWS CLI configuration
- Check IAM policies for required permissions
- Ensure user has access to all required AWS services

## Best Practices

1. **Always verify AWS account** before running Terraform commands
2. **Review plan carefully** before applying changes
3. **Use consistent naming** following the repository conventions
4. **Keep credentials secure** - never embed in configuration files
5. **Enable S3 versioning** for state file backup and recovery
6. **Test in development** environment before production deployment

## Next Steps

After successful deployment:
1. Configure your applications to use the deployed infrastructure
2. Set up monitoring using the CloudWatch alarms created
3. Configure DNS if using custom domains
4. Set up CI/CD pipelines for application deployment
5. Follow ongoing maintenance best practices

**Need help with specific issues?** See: `#troubleshooting-guide`