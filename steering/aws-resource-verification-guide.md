# AWS Resource Verification Guide for Kiro

This guide instructs Kiro to handle two distinct AWS account scenarios: brand new accounts (no validation needed) and existing accounts (full validation required).

> **Purpose**: Provide appropriate AWS resource analysis based on the user's account situation - skip validation for fresh accounts, perform comprehensive analysis for existing accounts.

---

## Overview

### Two-Path Approach

**Kiro should first determine the user's AWS account situation:**

1. **🆕 Brand New Account**: Fresh AWS account with no existing resources
   - ✅ Skip resource validation
   - ✅ Use default configuration
   - ✅ Proceed directly to deployment

2. **🏗️ Existing Account**: AWS account with existing resources
   - ✅ Perform comprehensive resource analysis
   - ✅ Detect conflicts and suggest resolutions
   - ✅ Generate adjusted configurations

### Decision Point

**Kiro should ask:**
```
🔧 AWS Account Setup

I need to understand your AWS account situation:

1. 🆕 **Brand New Account** - Fresh AWS account with no existing resources
2. 🏗️ **Existing Account** - AWS account with existing resources

Which option describes your situation?
```

---

## Path 1: Brand New Account (No Validation)

### What Kiro Should Do

**For fresh AWS accounts:**

1. **Basic Credential Verification Only:**
   ```bash
   # Simple verification - no resource scanning needed
   aws sts get-caller-identity --profile [selected-profile]
   ```

2. **Use Default Configuration:**
   ```hcl
   # Safe defaults for fresh accounts
   vpc_cidr_block = "10.0.0.0/16"
   application = "[user-specified]"
   environment = "[user-specified]"
   ```

3. **Skip All Resource Scanning:**
   - No VPC analysis needed
   - No database scanning required
   - No application resource checking
   - No storage analysis necessary

4. **Inform User:**
   ```
   ✅ Fresh AWS Account - No Validation Needed
   
   Since this is a brand new account, I'll use the optimal default configuration:
   • VPC CIDR: 10.0.0.0/16
   • Application: [user-specified]
   • Environment: [user-specified]
   
   No resource conflicts expected. Proceeding with S3 backend setup...
   ```

### Benefits of This Path

- ⚡ **Faster Setup**: No time spent on unnecessary resource scanning
- 🎯 **Optimal Configuration**: Uses best-practice defaults without conflicts
- 🚀 **Immediate Deployment**: Skip directly to S3 backend and Terraform setup
- 💡 **User-Friendly**: No complex analysis reports for simple scenarios

---

## Path 2: Existing Account (Full Validation)

### What Kiro Should Do

**For accounts with existing resources, perform comprehensive analysis:**

### Step 1: Verify AWS Access

```bash
# Verify credentials and get account information
aws sts get-caller-identity --profile [user-specified-profile]
```

**Expected Information:**
- AWS Account ID
- User ARN
- Region access
- Profile validation

### Step 2: Network Resource Analysis

```bash
# Get all VPCs and their CIDR blocks
aws ec2 describe-vpcs --profile [profile] --region [region] --query 'Vpcs[*].{VpcId:VpcId,CidrBlock:CidrBlock,State:State,IsDefault:IsDefault,Tags:Tags}'

# Get subnet information
aws ec2 describe-subnets --profile [profile] --region [region] --query 'Subnets[*].{SubnetId:SubnetId,VpcId:VpcId,CidrBlock:CidrBlock,AvailabilityZone:AvailabilityZone}'

# Check NAT gateways
aws ec2 describe-nat-gateways --profile [profile] --region [region] --query 'NatGateways[?State==`available`].{NatGatewayId:NatGatewayId,VpcId:VpcId,SubnetId:SubnetId,State:State}'
```

**Analysis Focus:**
- **CIDR Conflicts**: Check if default `10.0.0.0/16` overlaps with existing VPCs
- **VPC Count**: Ensure account isn't approaching VPC limits (5 per region default)
- **Subnet Distribution**: Check availability zone usage

### Step 3: Database Resource Analysis

```bash
# RDS Clusters
aws rds describe-db-clusters --profile [profile] --region [region] --query 'DBClusters[*].{ClusterIdentifier:DBClusterIdentifier,Engine:Engine,Status:Status,Endpoint:Endpoint,Port:Port}'

# ElastiCache Clusters
aws elasticache describe-cache-clusters --profile [profile] --region [region] --query 'CacheClusters[*].{ClusterId:CacheClusterId,Engine:Engine,Status:CacheClusterStatus,NodeType:CacheNodeType}'

# DocumentDB Clusters
aws docdb describe-db-clusters --profile [profile] --region [region] --query 'DBClusters[*].{ClusterIdentifier:DBClusterIdentifier,Engine:Engine,Status:Status,Endpoint:Endpoint,Port:Port}'
```

**Analysis Focus:**
- **Naming Patterns**: Check for similar database names
- **Engine Types**: Identify existing database engines
- **Resource Limits**: Check against service limits

### Step 4: Application Resource Analysis

```bash
# ECS Clusters
aws ecs list-clusters --profile [profile] --region [region]
aws ecs describe-clusters --profile [profile] --region [region] --clusters [cluster-arns] --query 'clusters[*].{ClusterName:clusterName,Status:status,RunningTasks:runningTasksCount,ActiveServices:activeServicesCount}'

# Application Load Balancers
aws elbv2 describe-load-balancers --profile [profile] --region [region] --query 'LoadBalancers[?Type==`application`].{Name:LoadBalancerName,DNSName:DNSName,State:State.Code,Scheme:Scheme}'

# ECR Repositories
aws ecr describe-repositories --profile [profile] --region [region] --query 'repositories[*].{Name:repositoryName,URI:repositoryUri,CreatedAt:createdAt}'
```

**Analysis Focus:**
- **ECS Cluster Names**: Check for naming conflicts with planned deployment
- **ALB Usage**: Identify existing load balancers
- **ECR Repositories**: Check for existing container repositories

### Step 5: Storage Resource Analysis

```bash
# S3 Buckets (global, but check for naming patterns)
aws s3api list-buckets --profile [profile] --query 'Buckets[*].{Name:Name,CreationDate:CreationDate}'

# EFS File Systems
aws efs describe-file-systems --profile [profile] --region [region] --query 'FileSystems[*].{FileSystemId:FileSystemId,CreationToken:CreationToken,LifeCycleState:LifeCycleState,PerformanceMode:PerformanceMode}'
```

### Step 6: Conflict Detection Logic

#### CIDR Block Conflict Detection

```bash
# Get all existing CIDR blocks
EXISTING_CIDRS=$(aws ec2 describe-vpcs --profile [profile] --region [region] --query 'Vpcs[*].CidrBlock' --output text)

# Check if default 10.0.0.0/16 conflicts
DEFAULT_CIDR="10.0.0.0/16"
```

**Conflict Resolution Logic:**
1. **No Conflict**: If no existing VPCs use `10.0.0.0/16`, proceed with default
2. **Conflict Detected**: If `10.0.0.0/16` exists, suggest alternatives:
   - `10.1.0.0/16`
   - `10.2.0.0/16`
   - `172.16.0.0/16`

#### Naming Conflict Detection

```bash
# Check if application name conflicts with existing resources
APPLICATION_NAME="[user-specified-app-name]"
ENVIRONMENT="[user-specified-environment]"

# Check ECS clusters for similar names
aws ecs list-clusters --profile [profile] --region [region] --query "clusterArns[?contains(@, '${APPLICATION_NAME}') || contains(@, '${ENVIRONMENT}')]"

# Check RDS clusters for similar names
aws rds describe-db-clusters --profile [profile] --region [region] --query "DBClusters[?contains(DBClusterIdentifier, '${APPLICATION_NAME}') || contains(DBClusterIdentifier, '${ENVIRONMENT}')].DBClusterIdentifier"
```

### Step 7: Present Analysis Results

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
   • ALBs: 2

⚠️  Conflicts Detected: 1

🔴 CIDR Conflicts:
   • Existing VPC uses 10.0.0.0/16 (default for new deployment)
   • Recommendation: Use 10.1.0.0/16 for new VPC

💡 Recommendations:
   • Adjust vpc_cidr_block to "10.1.0.0/16"
   • Proceed with medium setup (no resource limit issues)

✅ Safe to Deploy: YES (with adjustments)

Would you like me to apply these adjustments? (y/n)
============================================================
```

### Step 8: Apply Configuration Adjustments

```hcl
# Example: Conflict-free configuration adjustments
vpc_cidr_block = "10.1.0.0/16"  # Changed from default 10.0.0.0/16
application = "web-v2"           # Changed from "web" if conflicts found
environment = "dev-new"          # Changed from "dev" if conflicts found

# Availability zone adjustment if issues detected
availability_zones = ["us-east-1a", "us-east-1c", "us-east-1d"]  # Skip problematic AZs
```

---

## Integration with Deployment Workflow

### Pre-Deployment Decision Tree

```
User Request for ECS Deployment
         ↓
    Ask Account Type
         ↓
    ┌─────────────────┐
    │  Account Type?  │
    └─────────────────┘
         ↓
    ┌─────────┬─────────┐
    │ New     │ Existing│
    │ Account │ Account │
    └─────────┴─────────┘
         ↓         ↓
    Skip Analysis  Full Analysis
         ↓         ↓
    Default Config Adjusted Config
         ↓         ↓
    ┌─────────────────┐
    │ Proceed with    │
    │ S3 Backend      │
    │ Setup           │
    └─────────────────┘
```

### Error Handling for Both Paths

**Common Error Handling:**
```bash
# Check if AWS CLI is installed
if ! command -v aws &> /dev/null; then
    echo "❌ AWS CLI is not installed. Please install it first."
    exit 1
fi

# Check if profile exists
if ! aws configure list-profiles | grep -q "^${PROFILE}$"; then
    echo "❌ AWS profile '${PROFILE}' not found."
    echo "💡 Configure it with: aws configure --profile ${PROFILE}"
    exit 1
fi

# Check if credentials are valid
if ! aws sts get-caller-identity --profile ${PROFILE} &> /dev/null; then
    echo "❌ AWS credentials for profile '${PROFILE}' are invalid or expired."
    echo "💡 Reconfigure with: aws configure --profile ${PROFILE}"
    exit 1
fi
```

### MCP Server Integration

**Both paths use MCP servers:**
- **Terraform MCP Server**: For AWS CLI operations
- **Filesystem MCP Server**: For reading/writing configuration files
- **Git MCP Server**: For version control of configuration changes

---

## Summary

This two-path approach provides:

- ✅ **Efficient Fresh Account Setup**: No unnecessary validation for new accounts
- ✅ **Comprehensive Existing Account Analysis**: Full conflict detection for complex environments
- ✅ **User-Driven Choice**: Let users specify their situation
- ✅ **Optimal User Experience**: Fast path for simple cases, thorough analysis for complex cases
- ✅ **Smart Defaults**: Use best practices without conflicts

**Key Benefits:**
- **Faster deployment** for fresh accounts
- **Conflict prevention** for existing accounts
- **User-friendly approach** with clear choices
- **Leverages existing AWS credentials** through MCP servers