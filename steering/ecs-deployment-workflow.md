# ECS Infrastructure Deployment Workflow

This guide walks you through deploying ECS infrastructure using the battle-tested poc-idp repository with Terraform.

> **For complex deployments or structured planning**, see `#ai-dlc-infrastructure-workflow`

## Workflow Overview

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│  Requirements   │ → │     Design      │ → │     Tasks       │ → │ Implementation  │
│     Phase       │    │     Phase       │    │     Phase       │    │     Phase       │
└─────────────────┘    └─────────────────┘    └─────────────────┘    └─────────────────┘
```

**Repository cloning only happens during Implementation Phase.**

---

## Prerequisites

Before starting, ensure you have:
- AWS CLI configured (`aws sts get-caller-identity`)
- Terraform 1.0+ installed (`terraform --version`)
- Git installed (`git --version`)
- S3 bucket for Terraform state (setup below)

---

## Step 1: Setup Size Selection

Choose your infrastructure size based on requirements:

### Quick Decision Matrix

| Requirement | Small | Medium | Large |
|-------------|:-----:|:------:|:-----:|
| High Availability | ❌ | ⚠️ Partial | ✅ Full |
| Caching Layer | ❌ | ✅ Basic | ✅ Advanced |
| Document Database | ❌ | ✅ Basic | ✅ Clustered |
| Backup Strategy | ❌ | ✅ Comprehensive | ✅ Enterprise |
| VPN Access | ❌ | ❌ | ✅ |
| Production Ready | ❌ | ⚠️ Staging | ✅ |

### Setup Descriptions

**Small Setup** (`tfvars-sample/example-small-setup.tfvars`)
- **Cost:** $50-100/month, **Services:** <5
- **Network:** VPC, single NAT, bastion host
- **Database:** RDS Aurora (1 instance, no MultiAZ)
- **Application:** ECS, up to 5 services, Fargate
- **Best For:** Development, testing, MVPs

**Medium Setup** (`tfvars-sample/example-medium-setup.tfvars`)
- **Cost:** $200-400/month, **Services:** 5-13
- **Network:** VPC + private endpoints
- **Database:** RDS Aurora (2 instances), ElastiCache, DocumentDB
- **Application:** ECS, up to 13 services, Fargate
- **Monitoring:** CloudWatch + SNS notifications
- **Backups:** AWS Backup enabled
- **Best For:** Staging, small production

**Large Setup** (`tfvars-sample/example-large-setup.tfvars`)
- **Cost:** $800-1500/month, **Services:** 13-23
- **Network:** Full HA NAT, Client VPN, VPC flow logs
- **Database:** RDS Serverless V2 (MultiAZ), ElastiCache (MultiAZ), DocumentDB cluster
- **Application:** ECS, up to 23 services, Fargate with autoscaling
- **Monitoring:** Comprehensive CloudWatch + SNS
- **Backups:** Full AWS Backup coverage
- **Best For:** Production, mission-critical workloads

---

## Step 2: S3 Backend Setup

Set up Terraform state storage before deployment.

### Automated S3 Backend Setup

**Kiro should use developer's AWS credentials directly through MCP servers** (see `#aws-resource-verification-guide` for complete instructions).

**S3 Backend Setup Process:**
1. ✅ Verify AWS credentials and permissions
2. ✅ Check if S3 bucket exists and is properly configured
3. ✅ Create bucket with security best practices if needed
4. ✅ Configure versioning, encryption, and public access blocking
5. ✅ Create DynamoDB table for state locking
6. ✅ Generate Terraform backend configuration file

**Key Commands for Kiro to Execute:**

### Option A: Use Existing Bucket

**Kiro should validate existing bucket:**
```bash
# Check if bucket exists and is accessible
aws s3 ls s3://your-terraform-state-bucket --profile [profile]

# Check versioning status
aws s3api get-bucket-versioning --bucket your-terraform-state-bucket --profile [profile]

# Check encryption status
aws s3api get-bucket-encryption --bucket your-terraform-state-bucket --profile [profile]

# Check public access block
aws s3api get-public-access-block --bucket your-terraform-state-bucket --profile [profile]
```

**If configuration issues found, Kiro should fix them:**
```bash
# Enable versioning if disabled
aws s3api put-bucket-versioning \
  --bucket your-terraform-state-bucket \
  --versioning-configuration Status=Enabled \
  --profile [profile]

# Enable encryption if missing
aws s3api put-bucket-encryption \
  --bucket your-terraform-state-bucket \
  --server-side-encryption-configuration '{
    "Rules": [{"ApplyServerSideEncryptionByDefault": {"SSEAlgorithm": "AES256"}}]
  }' \
  --profile [profile]

# Block public access if not configured
aws s3api put-public-access-block \
  --bucket your-terraform-state-bucket \
  --public-access-block-configuration '{
    "BlockPublicAcls": true,
    "IgnorePublicAcls": true,
    "BlockPublicPolicy": true,
    "RestrictPublicBuckets": true
  }' \
  --profile [profile]
```

### Option B: Create New Backend

**Kiro should create complete new backend:**
```bash
# Create bucket (handle us-east-1 special case)
if [ "[region]" = "us-east-1" ]; then
  aws s3 mb s3://your-terraform-state-bucket --profile [profile]
else
  aws s3 mb s3://your-terraform-state-bucket --region [region] --profile [profile]
fi

# Enable versioning
aws s3api put-bucket-versioning \
  --bucket your-terraform-state-bucket \
  --versioning-configuration Status=Enabled \
  --profile [profile]

# Enable encryption
aws s3api put-bucket-encryption \
  --bucket your-terraform-state-bucket \
  --server-side-encryption-configuration '{
    "Rules": [{"ApplyServerSideEncryptionByDefault": {"SSEAlgorithm": "AES256"}}]
  }' \
  --profile [profile]

# Block public access
aws s3api put-public-access-block \
  --bucket your-terraform-state-bucket \
  --public-access-block-configuration '{
    "BlockPublicAcls": true,
    "IgnorePublicAcls": true,
    "BlockPublicPolicy": true,
    "RestrictPublicBuckets": true
  }' \
  --profile [profile]

# Create DynamoDB table for state locking
aws dynamodb create-table \
  --table-name terraform-state-lock \
  --attribute-definitions AttributeName=LockID,AttributeType=S \
  --key-schema AttributeName=LockID,KeyType=HASH \
  --billing-mode PAY_PER_REQUEST \
  --region [region] \
  --profile [profile]
```

### Create Backend Configuration

**Kiro should generate backend configuration file:**

Create `tfbackends/[application]-[account_id]-[region].tfbackend`:
```hcl
bucket         = "your-terraform-state-bucket"
key            = "terraform/[application]/[environment]/terraform.tfstate"
region         = "[region]"
encrypt        = true
dynamodb_table = "terraform-state-lock"
```

**Expected Output:**
```
🚀 Setting up Terraform S3 backend...
✅ Connected to AWS account: 253650698585
📦 Creating S3 bucket: my-terraform-state
✅ Bucket created successfully
🔄 Enabling versioning...
✅ Versioning enabled
🔐 Enabling encryption...
✅ Encryption enabled
🚫 Blocking public access...
✅ Public access blocked
🗄️ Creating DynamoDB table: terraform-state-lock
✅ Table created successfully
📝 Backend configuration written to: tfbackends/myapp-253650698585-us-east-1.tfbackend

🎉 S3 backend setup completed successfully!
```

---

## Step 3: Implementation

**Only after spec workflow is complete (requirements.md, design.md, tasks.md approved).**

### Clone Repository

**Via Git MCP:** Clone the poc-idp repository:

```bash
# Clone to temporary directory
git clone https://github.com/santyar0/poc-idp.git temp-poc-idp

# Merge into current directory
mv temp-poc-idp/.git ./
mv temp-poc-idp/* ./
rm -rf temp-poc-idp
```

**Repository URL:** `https://github.com/santyar0/poc-idp.git`

**Automated:** The agent can perform git operations using Git MCP server.

### Step 4: AWS Account Setup Choice

**Kiro should ask the user to choose their AWS account situation:**

```
🔧 AWS Account Setup

I need to understand your AWS account situation to proceed correctly:

1. 🆕 **Brand New Account** - I have a fresh AWS account with no existing resources
2. 🏗️ **Existing Account** - I have an AWS account with existing resources that need validation

Which option describes your situation?
```

#### Option 1: Brand New Account (No Validation Needed)

**For fresh AWS accounts, Kiro should:**

1. **Verify Basic Access:**
   ```bash
   # Simple credential verification
   aws sts get-caller-identity --profile [selected-profile]
   ```

2. **Proceed with Default Configuration:**
   - Use default VPC CIDR: `10.0.0.0/16`
   - Use user-specified application and environment names
   - No conflict checking needed
   - Skip resource scanning

3. **Inform User:**
   ```
   ✅ Fresh AWS Account Detected
   
   Since this is a brand new account, I'll use the default configuration:
   • VPC CIDR: 10.0.0.0/16
   • Application: [user-specified]
   • Environment: [user-specified]
   
   No resource conflicts expected. Proceeding with deployment setup...
   ```

#### Option 2: Existing Account (Full Validation Required)

**For accounts with existing resources, Kiro should perform comprehensive analysis** (see `#aws-resource-verification-guide` for complete instructions):

1. **Verify AWS Credentials:**
   ```bash
   aws sts get-caller-identity --profile [selected-profile]
   ```

2. **Comprehensive Resource Scanning:**
   ```bash
   # Network analysis
   aws ec2 describe-vpcs --profile [profile] --region [region] --query 'Vpcs[*].{VpcId:VpcId,CidrBlock:CidrBlock,State:State}'
   
   # Database analysis
   aws rds describe-db-clusters --profile [profile] --region [region] --query 'DBClusters[*].{ClusterIdentifier:DBClusterIdentifier,Engine:Engine,Status:Status}'
   
   # Application analysis
   aws ecs list-clusters --profile [profile] --region [region]
   aws elbv2 describe-load-balancers --profile [profile] --region [region] --query 'LoadBalancers[?Type==`application`]'
   
   # Storage analysis
   aws s3api list-buckets --profile [profile] --query 'Buckets[*].{Name:Name,CreationDate:CreationDate}'
   ```

3. **Conflict Detection and Resolution:**
   
   **CIDR Block Analysis:**
   ```bash
   # Check for VPC CIDR conflicts
   EXISTING_CIDRS=$(aws ec2 describe-vpcs --profile [profile] --region [region] --query 'Vpcs[*].CidrBlock' --output text)
   
   # If 10.0.0.0/16 exists, recommend 10.1.0.0/16 or 10.2.0.0/16
   ```
   
   **Naming Conflict Analysis:**
   ```bash
   # Check ECS clusters for naming conflicts
   aws ecs list-clusters --profile [profile] --region [region] --query "clusterArns[?contains(@, '${APPLICATION_NAME}')]"
   
   # Check RDS clusters for naming conflicts
   aws rds describe-db-clusters --profile [profile] --region [region] --query "DBClusters[?contains(DBClusterIdentifier, '${APPLICATION_NAME}')]"
   ```

4. **Present Analysis Results:**
   ```
   🔍 AWS ACCOUNT ANALYSIS SUMMARY
   ============================================================
   📋 Account ID: 253650698585
   👤 Profile: default
   🌍 Region: us-east-1
   
   📊 Resource Summary:
      • VPCs: 2 (CIDR: 10.0.0.0/16, 172.31.0.0/16)
      • RDS Clusters: 1 (mysql-prod-cluster)
      • ECS Clusters: 0
      • S3 Buckets: 15
   
   ⚠️  Conflicts Detected: 1
      CIDR Conflicts:
        • Existing VPC uses 10.0.0.0/16 (conflicts with default)
   
   💡 Recommendations:
      • Use CIDR block 10.1.0.0/16 for new VPC
      • Proceed with deployment using adjusted configuration
   
   Would you like me to apply these adjustments? (y/n)
   ```

5. **Apply Configuration Adjustments:**
   ```hcl
   # Adjusted tfvars based on analysis
   vpc_cidr_block = "10.1.0.0/16"  # Adjusted to avoid existing 10.0.0.0/16
   application = "web-v2"           # Adjusted if naming conflicts found
   environment = "dev-new"          # Adjusted if conflicts detected
   ```

#### AWS Profile Selection

**For both options, Kiro should first ask:**

```
🔑 AWS Profile Selection

Which AWS CLI profile should I use?

Available profiles:
• default
• production  
• staging
• development

Or specify a custom profile name: ___________
```

**Profile Verification:**
```bash
# Test selected profile
aws sts get-caller-identity --profile [selected-profile]

# If successful, show account info
# If failed, ask user to configure: aws configure --profile [profile-name]
```

### Step 5: Interactive Configuration Collection

Collect configuration through interactive dialog. User can provide values or use defaults:

#### Required Configuration Fields

**AWS Account Settings:**
```
account_id = "253650698585"  # CHANGE_ME_PLEASE - AWS account ID where all resources should be configured
region = "us-east-1"         # CHANGE_ME_PLEASE - Region in AWS cloud where all resources should be configured
```

**Application Settings:**
```
application = "web"          # CHANGE_ME_PLEASE - Name of the application
environment = "dev"          # CHANGE_ME_PLEASE - Environment name (dev/staging/prod)
```

**Optional Settings:**
```
# additional_tags = {         # Optional - The list of additional tags
#   product = "product_name"
#   project = "company_name"
# }
```

#### Interactive Collection Process

For each field, prompt user with:
- **Field description** and current default value
- **Option to skip** (use default) or provide custom value
- **Validation** for required formats (account_id: 12 digits, region: valid AWS region)

### Step 6: Create Project Configuration

1. **Copy template** from `tfvars-sample/example-{size}-setup.tfvars` to `tfvars/[application]-[account_id]-[region].tfvars`
2. **Apply collected values** to the copied configuration file
3. **Validate configuration** before proceeding

#### File Structure

```
tfvars-sample/                    # Template files (read-only)
├── example-small-setup.tfvars
├── example-medium-setup.tfvars
└── example-large-setup.tfvars

tfvars/                          # Generated configuration files
└── [application]-[account_id]-[region].tfvars

tfbackends/                      # Backend configuration files
└── [application]-[account_id]-[region].tfbackend
```

#### Configuration Process

1. **Select setup size** (small/medium/large)
2. **Collect interactive configuration** (account_id, region, application, environment, optional tags)
3. **Copy template** from `tfvars-sample/` to `tfvars/`
4. **Apply collected values** to the copied file
5. **Create backend configuration** in `tfbackends/`
| Large | `tfvars/example-large-setup.tfvars` |

**Mandatory variables to update:**
```hcl
environment = "[environment]"
account_id  = "[account_id]"
region      = "[region]"
application = "[application]"
```

**Configuration values are automatically applied from the interactive collection process.**

### Step 7: Verify AWS Access

**Via Terraform MCP:** Verify AWS credentials and access:

```bash
aws sts get-caller-identity
```

**Validation checks:**
- Account ID matches collected value
- User has required permissions (ECS, VPC, RDS, S3, IAM)
- Region matches collected value

**Automated:** The agent can verify AWS access through MCP servers.

### Step 8: Initialize Terraform

**Via Terraform MCP:** Initialize Terraform with backend configuration:

```bash
terraform init \
  -backend-config=tfbackends/[application]-[account_id]-[region].tfbackend \
  -reconfigure
```

### Step 9: Review Plan

**Via Terraform MCP:** Generate and review Terraform plan:

```bash
terraform plan -var-file=tfvars/[application]-[account_id]-[region].tfvars
```

**Review output for:**
- Resources to be created
- Configuration issues
- Cost implications

**If modifications needed:** Update tfvars and repeat plan.

**Automated:** The agent can generate and analyze Terraform plans through Terraform MCP server.

### Step 10: Deploy

**Via Terraform MCP:** Apply the Terraform configuration:

```bash
terraform apply -var-file=tfvars/[application]-[account_id]-[region].tfvars
```

**Automated:** The agent can execute Terraform apply through Terraform MCP server.

### Step 11: Verify Deployment

**Via Terraform MCP:** Post-deployment verification:
- [ ] ECS cluster running and healthy
- [ ] ALB health checks passing
- [ ] Database connectivity (if configured)
- [ ] Endpoints accessible

**Automated:** The agent can verify deployment status through Terraform MCP server.

---

## MCP Automation Summary

The following operations can be **automated through MCP servers using developer's AWS credentials**:

| Operation | MCP Server | Automation Level |
|-----------|------------|------------------|
| **AWS credential verification** | Terraform MCP | ✅ Direct credential validation |
| **AWS resource scanning** | Terraform MCP | ✅ Comprehensive automated analysis |
| **Conflict detection** | Terraform MCP | ✅ Intelligent conflict resolution |
| **Configuration adjustment** | Filesystem MCP | ✅ Automated config generation |
| **S3 backend setup** | Terraform MCP | ✅ Complete backend automation |
| **DynamoDB table creation** | Terraform MCP | ✅ Automated with validation |
| **Repository cloning** | Git MCP | ✅ Fully automated |
| **File creation/copying** | Filesystem MCP | ✅ Fully automated |
| **Configuration generation** | Filesystem MCP | ✅ Template-based automation |
| **Terraform init/plan/apply** | Terraform MCP | ✅ Fully automated |

## Direct AWS Integration Benefits

**Key Advantages of Direct AWS CLI Integration:**
- ✅ **No Additional Scripts**: Uses existing AWS CLI and MCP servers
- ✅ **Developer Credentials**: Leverages user's existing AWS access
- ✅ **Real-time Analysis**: Direct AWS API calls for current state
- ✅ **MCP Integration**: Seamless integration with existing workflow
- ✅ **Cross-Platform**: Works on Windows, macOS, and Linux
- ✅ **Immediate Results**: No script installation or setup required

**Kiro's Analysis Capabilities:**
- **Network Analysis**: VPC CIDR conflict detection and resolution
- **Resource Scanning**: Comprehensive AWS resource inventory
- **Conflict Prevention**: Intelligent naming and configuration adjustments
- **Security Validation**: S3 bucket security configuration verification
- **Limit Checking**: AWS service limit validation and warnings

**Integration Points:**
- `#aws-resource-verification-guide` - Complete analysis instructions for Kiro
- Terraform MCP Server - AWS CLI command execution
- Filesystem MCP Server - Configuration file generation
- Git MCP Server - Version control integration

## Resource Conflict Prevention

**Automated Scanning:** The workflow now includes comprehensive AWS account scanning to:
- **Detect existing resources** (VPCs, RDS, ECS, etc.)
- **Identify naming conflicts** and suggest alternatives
- **Avoid CIDR block overlaps** in network configuration
- **Prevent resource duplication** while maintaining isolation
- **Check account limits** and warn of potential issues

**Smart Configuration:** Based on scan results, automatically adjusts:
- VPC CIDR blocks to avoid conflicts
- Resource naming to prevent duplicates
- Availability zone selection if needed
- Application naming if conflicts detected

## Command Reference

**Automated execution with Kiro (recommended):**

```bash
# Kiro performs comprehensive AWS analysis using developer credentials
# See #aws-resource-verification-guide for complete instructions

# Analysis and backend setup through MCP servers
aws sts get-caller-identity --profile [profile]
aws ec2 describe-vpcs --profile [profile] --region [region]
aws s3 mb s3://[bucket-name] --region [region] --profile [profile]

# Complete deployment sequence with Terraform
terraform init -backend-config=tfbackends/[app]-[account]-[region].tfbackend -reconfigure
terraform plan -var-file=tfvars/[app]-[account]-[region].tfvars
terraform apply -var-file=tfvars/[app]-[account]-[region].tfvars
```

**Manual execution (if needed):**

```bash
# Complete deployment sequence
aws sts get-caller-identity
terraform init -backend-config=tfbackends/[app]-[account]-[region].tfbackend -reconfigure
terraform plan -var-file=tfvars/[app]-[account]-[region].tfvars
terraform apply -var-file=tfvars/[app]-[account]-[region].tfvars

# Useful operations
terraform plan -target=aws_ecs_service.app    # Target specific resource
terraform apply -parallelism=20               # Faster deployment
terraform output                              # Show outputs
terraform state list                          # List managed resources
```

---

## Next Steps

After deployment:
1. Configure applications for deployed infrastructure
2. Set up monitoring with created CloudWatch alarms
3. Configure DNS for custom domains
4. Set up CI/CD pipelines

---

## Related Guides

- `#component-extension-guide` - Adding new components
- `#poc-idp-style-guide` - Repository coding patterns and best practices
- `#troubleshooting-guide` - Common issues and solutions
- `#ai-dlc-infrastructure-workflow` - Structured planning methodology
